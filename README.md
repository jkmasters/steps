# steps

Treadmill walking data from a KingSmith WalkingPad A1 Pro, published automatically
(every ~10 min while there's new data) by a logger on Jake's Mac.

## Fetching

```
https://raw.githubusercontent.com/jkmasters/steps/main/data/v1/steps.json
```

Served with `Access-Control-Allow-Origin: *`, so any site can `fetch()` it directly.
GitHub caches it for up to ~5 minutes.

```js
const data = await fetch("https://raw.githubusercontent.com/jkmasters/steps/main/data/v1/steps.json")
  .then(r => r.json());
```

## Schema (v1)

The path is versioned: breaking changes go to `data/v2/`, and `v1` keeps its shape. If the
storage backend changes, this schema is the contract to keep.

```jsonc
{
  "schema_version": 1,
  "timezone": "America/New_York",            // IANA zone the dates below are in
  "updated_at": "2026-10-02T15:17:31-04:00", // latest reading from the pad; null if no data
  "first_day": "2026-10-02",                 // first day with logging; null if no data
  "totals": { "steps": 8643, "distance_m": 6740, "belt_time_s": 5900, "walks": 1 },
  "race": {                                  // walking due west from Atlanta to the Pacific
    "name": "Atlanta to the Pacific",
    "route": "due west along 33.749° N",
    "start":  { "name": "Downtown Atlanta, Georgia", "lat": 33.749, "lon": -84.388 },
    "finish": { "name": "Long Beach, California",    "lat": 33.749, "lon": -118.10974 },
    "goal_mi": 1939.0,
    "distance_mi": 9.22,                     // all-time treadmill distance
    "progress_pct": 0.476,                   // not capped: exceeds 100 after finishing
    "remaining_mi": 1929.78,                 // 0 once finished
    "position": { "lat": 33.749, "lon": -84.54837, "place": "South Fulton, Georgia" },
    "map_url": "https://www.google.com/maps/search/?api=1&query=33.74900,-84.54837",
    "finished_on": null                      // YYYY-MM-DD of the day the goal was passed
  },
  "days": [                                  // first_day .. last day walked, gaps filled with 0
    { "date": "2026-10-02", "steps": 8643, "distance_m": 6740, "belt_time_s": 5900, "walks": 1 }
  ],
  "sessions": [                              // one per walk, oldest first
    {
      "started_at": "2026-10-02T13:39:13-04:00", // estimated: first reading minus belt time
      "ended_at": "2026-10-02T15:17:31-04:00",   // latest reading (still growing if walk is live)
      "steps": 8643,
      "distance_m": 6740,                        // pad resolution is 10 m
      "belt_time_s": 5900,                       // time the belt was moving
      "max_speed_kmh": 4.8,
      "avg_hr_bpm": 108                          // null if no heart-rate sensor was worn
    }
  ]
}
```

Notes for consumers:
- A day is assigned by the walk's local start date.
- `days` ends at the last day walked; days after it (up to today) are zero.
- A walk in progress shows up as the last session with a recent `ended_at`.
- `avg_hr_bpm` averages 5 s heart-rate samples taken while the belt was moving (pauses
  excluded). Added 2026-10-02; older walks have `null`.
- `race` (added 2026-10-02) places the all-time distance on a due-west line from downtown Atlanta;
  one degree of longitude = 57.5 mi. `position.place` is a city, or a county in rural areas
  (OpenStreetMap), and `null` if the lookup failed. Past the finish the walk continues west, so
  `place` becomes the Pacific Ocean. The goal may be re-set after someone finishes.

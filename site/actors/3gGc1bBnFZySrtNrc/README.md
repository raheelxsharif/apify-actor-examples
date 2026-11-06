# Google Maps Timezone & Local Time Lookup

Resolve the IANA timezone, current local time, UTC offset and daylight-saving status for any latitude/longitude on Earth — a single coordinate or a batch of them in one run. Useful whenever you have geocoded data and need to know what time it actually is at that point on the map right now.

## What this actor does

- Resolves a single `{lat, lng}` coordinate, or a batch of many, to its IANA timezone name
- Returns the current local wall-clock time and the equivalent UTC time for each coordinate
- Reports the current UTC offset (both formatted and in seconds) and whether daylight saving time is currently active
- Includes daylight-saving transition details — standard offset, DST offset, and start/end dates — when the source has that data for the zone
- Leaves a field out of the record entirely when no valid value is available, rather than guessing a timezone or offset

## Output per location

- `inputLocation` — the exact `{lat, lng}` coordinate you submitted
- `timezone` — IANA zone name (e.g. `America/New_York`, `Asia/Tokyo`, `Australia/Sydney`)
- `timezoneAbbreviation` — short zone code when one applies and DST status is known (e.g. `EDT`, `BST`)
- `utcOffset` — current offset formatted as `+HH:MM` / `-HH:MM`
- `utcOffsetSeconds` — current offset in seconds (positive east of UTC)
- `currentLocalTime` — wall-clock time at the coordinate, ISO 8601
- `currentUtcTime` — the equivalent UTC time, ISO 8601
- `dayOfWeek` — day of the week at the coordinate's local time, when available
- `unixTimestamp` — Unix timestamp in seconds, when available
- `isDaylightSavingTime` — `true` when daylight saving time is currently in effect
- `daylightSavings` — an object with `inEffect` and `observed`, plus (when known) `nextTransition`, `dstName`, `standardUtcOffset`, `standardUtcOffsetSeconds`, `dstOffsetSeconds`, `dstUtcOffsetSeconds`, `startsAt`, and `endsAt`
- `sourceUrl` — the exact lookup URL used to resolve this coordinate
- `recordType` — always `"timezone"`
- `scrapedAt` — UTC timestamp of when the lookup was performed

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `location` | object | – | A single `{lat, lng}` coordinate to resolve. Provide this or `locations` |
| `locations` | array | – | Multiple `{lat, lng}` coordinates to resolve in one run |
| `proxyConfiguration` | object | off | Optional Apify proxy configuration; not normally required |

At least one of `location` or `locations` must be provided. Latitude must fall within `[-90, 90]` and longitude within `[-180, 180]`; the keys `latitude`/`longitude`/`lon` are also accepted as synonyms for `lat`/`lng`.

### Example: a single coordinate

```json
{
  "location": { "lat": 27.7172, "lng": 85.3240 }
}
```

### Example: a worldwide batch

```json
{
  "locations": [
    { "lat": 40.7128, "lng": -74.0060 },
    { "lat": 51.5074, "lng": -0.1278 },
    { "lat": 35.6762, "lng": 139.6503 },
    { "lat": -33.8688, "lng": 151.2093 }
  ]
}
```

### Example: two cities to compare local time

```json
{
  "locations": [
    { "lat": 40.7128, "lng": -74.0060 },
    { "lat": 35.6762, "lng": 139.6503 }
  ]
}
```

## Use cases

- **Normalize business hours** — pair with a places or geocoding actor to convert local opening hours into UTC ranges
- **Schedule outreach across regions** — only trigger notifications during local business hours for each recipient
- **Enrich geocoded datasets** — attach a timezone and current local time to addresses or coordinates you already have
- **Travel and logistics planning** — check the local time and DST status at a destination before scheduling a call or delivery
- **Compliance and audit logging** — stamp events with the correct local time and DST flag for the location they occurred at

## FAQ

**Does this need a Google Maps API key?**
No. Timezone lookups work from latitude/longitude alone with no API key, login, or cookies required.

**Why are some fields missing from a record?**
The actor never invents a timezone, offset or DST detail. If the underlying time-zone directory doesn't return a particular value for a coordinate (for example, a DST abbreviation when the zone has no DST), that field is simply left out of the record.

**How accurate is the timezone near borders or disputed regions?**
The actor returns the IANA timezone identifier exactly as resolved for the coordinate, without inferring a zone from longitude alone. Boundary and disputed-area assignments reflect the underlying time-zone directory's own data. Fractional offsets (such as `+05:45`) are preserved in seconds.

**What happens if I submit an invalid coordinate?**
Out-of-range or non-numeric latitude/longitude values are rejected with a clear error before any lookup is attempted.

**Can I resolve a large batch of coordinates in one run?**
Yes — `locations` accepts many coordinates, processed one after another. For very large batches, test with a smaller sample first to confirm the result shape fits your pipeline.

**Does this return the country for a coordinate?**
No — country information isn't part of this lookup. Pair this actor with a geocoding actor if you need the country or address for the same coordinate.

**Is this affiliated with Google?**
No. This actor resolves timezone data independently from public coordinate data; it is not affiliated with or endorsed by Google.

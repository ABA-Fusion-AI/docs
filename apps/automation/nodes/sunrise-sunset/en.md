---
node_id: "sunrise-sunset"
title: "Sunrise-Sunset"
description: "Get sunrise and sunset times for a location using the Sunrise-Sunset API."
category: "Astronomy / Weather"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:

- sunrise
- sunset
- astronomy
- solar
- twilight
- location
- daylight
- api

related_nodes:
- function
- if
- http-request

---

# Sunrise-Sunset

> **Category:** astronomy-nodes | **Type:** Action Node

Get **sunrise, sunset, solar noon, and twilight times** for a given latitude/longitude using the public Sunrise-Sunset API.

The **Sunrise-Sunset** node queries a location's solar event times for today (default) or a specific date, and returns the full set of civil, nautical, and astronomical twilight boundaries alongside sunrise/sunset.

### Supported Features

- Sunrise and sunset time lookup by latitude/longitude
- Optional specific date lookup (defaults to today if omitted)
- Solar noon and day length
- Civil, nautical, and astronomical twilight begin/end times
- Required-field validation for `lat`/`lng`

### Use Cases

- Schedule automations relative to sunrise or sunset (e.g. outdoor lighting, irrigation)
- Display daylight information for a location in a dashboard or report
- Calculate day length trends across dates for a fixed location
- Trigger notifications at civil twilight for photography or outdoor event planning
- Feed solar times into a scheduling or `If` node for time-of-day logic

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `lat` | `number` | ✅ Yes | — | Latitude of the location. |
| `lng` | `number` | ✅ Yes | — | Longitude of the location. |
| `date` | `string` | ❌ No | `""` | Date to query, in `YYYY-MM-DD` format (or any format accepted by the Sunrise-Sunset API). Empty string means today. |

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `sunrise` | `string` | Sunrise time. |
| `sunset` | `string` | Sunset time. |
| `solar_noon` | `string` | Solar noon time. |
| `day_length` | `string` | Length of the day. |
| `civil_twilight_begin` | `string` | Start of civil twilight. |
| `civil_twilight_end` | `string` | End of civil twilight. |
| `nautical_twilight_begin` | `string` | Start of nautical twilight. |
| `nautical_twilight_end` | `string` | End of nautical twilight. |
| `astronomical_twilight_begin` | `string` | Start of astronomical twilight. |
| `astronomical_twilight_end` | `string` | End of astronomical twilight. |

All time values are returned exactly as provided by the Sunrise-Sunset API (typically in UTC, formatted as a time string, unless the API's `formatted=0` parameter is used — not applied by this node).

---

## Output Example

```json
{
  "success": true,
  "sunrise": "6:12:34 AM",
  "sunset": "7:45:21 PM",
  "solar_noon": "12:58:57 PM",
  "day_length": "13:32:47",
  "civil_twilight_begin": "5:45:10 AM",
  "civil_twilight_end": "8:12:45 PM",
  "nautical_twilight_begin": "5:13:02 AM",
  "nautical_twilight_end": "8:44:53 PM",
  "astronomical_twilight_begin": "4:41:18 AM",
  "astronomical_twilight_end": "9:16:37 PM"
}
```

---

## Configuration Examples

### Today's Times for a Location

```json
{
  "lat": 48.8566,
  "lng": 2.3522
}
```

### Specific Date

```json
{
  "lat": 48.8566,
  "lng": 2.3522,
  "date": "2026-12-21"
}
```

---

## Workflow Integration

### Sample Workflow: Fetch Times → Function

```json
{
  "nodes": [
    {
      "id": "sunrise-sunset",
      "type": "sunrise-sunset",
      "config": {
        "lat": 48.8566,
        "lng": 2.3522
      }
    },
    {
      "id": "compute-schedule",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Fetch Times → If → Notification

```json
{
  "nodes": [
    {
      "id": "sunrise-sunset",
      "type": "sunrise-sunset",
      "config": {
        "lat": 40.7128,
        "lng": -74.0060
      }
    },
    {
      "id": "check-near-sunset",
      "type": "if"
    },
    {
      "id": "notify-outdoor-team",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Daily Schedule → Sunrise-Sunset → Database

```json
{
  "nodes": [
    {
      "id": "sunrise-sunset",
      "type": "sunrise-sunset",
      "config": {
        "lat": 51.5074,
        "lng": -0.1278
      }
    },
    {
      "id": "log-daylight",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- Schedule (daily) → Sunrise-Sunset → Function → Trigger (lights on/off at sunrise/sunset)
- Sunrise-Sunset → Database → Chart/visualization pipeline — track day length trends over time
- Sunrise-Sunset → If (civil twilight window) → Notification — photography/event timing alerts

---

## Error Handling

### Missing Coordinates

```text
Latitude and longitude are required
```

Raised when `lat` or `lng` is falsy (including `0`, since the check is `!lat || !lng`).

### Sunrise-Sunset API Error (HTTP)

```text
Sunrise-Sunset API error: <status>
```

Raised when the API returns a non-OK HTTP status.

### Sunrise-Sunset API Error (Status Field)

```text
Sunrise-Sunset API error: <status>
```

Raised when the response is HTTP OK but the API's own `status` field is not `"OK"` (e.g. `"INVALID_REQUEST"`, `"INVALID_DATE"`).

### Wrapped Failure

```text
Sunrise-Sunset request failed: <underlying error message>
```

All errors, including both cases above, are re-thrown wrapped in this message from `handleTick`.

---

## Troubleshooting

### "Sunrise-Sunset request failed: Latitude and longitude are required"

**Cause**

`lat` or `lng` was left unset, or one of them was set to exactly `0` — the falsy check (`!lat || !lng`) treats `0` the same as missing.

**Solution**

Ensure both `lat` and `lng` are set to real, non-zero coordinate values. If a location genuinely sits on the equator or prime meridian (`0.0`), be aware this validation will incorrectly reject it — use a very small non-zero value as a workaround if needed (e.g. `0.0001`).

---

### "Sunrise-Sunset request failed: Sunrise-Sunset API error: INVALID_REQUEST"

**Cause**

`lat` or `lng` is outside the valid range (latitude: -90 to 90, longitude: -180 to 180), or malformed.

**Solution**

Verify the coordinate values are valid decimal degrees within range.

---

### "Sunrise-Sunset request failed: Sunrise-Sunset API error: INVALID_DATE"

**Cause**

The `date` parameter is not in a format the API accepts.

**Solution**

Use `YYYY-MM-DD` format (e.g. `"2026-12-21"`), or leave `date` empty to default to today.

---

### Times Don't Match Local Expectations

**Cause**

The Sunrise-Sunset API returns times in **UTC** by default — this node does not request or convert to local time.

**Solution**

Convert the returned times to the target location's local time zone downstream (e.g. in a `Function` node), accounting for the location's UTC offset and DST rules.

---

## Security

The node performs outbound HTTP requests to the public Sunrise-Sunset API (`api.sunrise-sunset.org`).

No API key or authentication credential is required.

---

## Notes

The node returns all solar event fields available from the API's default response — it does not request the `formatted=0` (ISO 8601 / UTC offset) response mode, so all times are returned as human-readable local-format strings in UTC.

The node does not:

- Convert returned times to the location's local time zone
- Support time zone or `formatted` API parameters
- Cache results between calls
- Validate `date` format before sending it to the API
- Support fetching a range of dates in a single call

It is intended to provide a straightforward solar-event lookup for a single location and date for downstream scheduling and reporting workflows.

---

## Related

- [Function](./function.md) – Convert times to local time zone or compute schedules
- [If](./if.md) – Route workflows based on time-of-day logic
- [HTTP Request](./http-request.md) – Make additional custom Sunrise-Sunset API calls (e.g. with `formatted=0`)

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-18 | Initial release |
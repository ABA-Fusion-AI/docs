---

node_id: "gdacs"
title: "GDACS"
description: "Get disaster alerts from the Global Disaster Alert and Coordination System (GDACS)."
category: "Disaster & Alerts"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - gdacs
  - disaster
  - disasters
  - disaster-alerts
  - emergency
  - alerts
  - earthquake
  - flood
  - tropical-cyclone
  - global
  - action
related_nodes:
  - function
  - if
  - http-request
  - notification

---

# GDACS

> **Category:** disaster-nodes | **Type:** Action Node

Get disaster alerts from the **Global Disaster Alert and Coordination System (GDACS)**.

The **GDACS** node retrieves disaster events from the GDACS API and returns structured information about events such as tropical cyclones, floods, and earthquakes.

The node supports filtering events by event type and alert level, as well as limiting the number of returned events.

### Supported Features

- Fetch disaster events from GDACS
- Retrieve tropical cyclone events
- Retrieve flood events
- Retrieve earthquake events
- Filter events by event type
- Filter events by alert level
- Limit the number of returned events
- Return event severity information
- Return affected country information
- Return population information
- Return vulnerability information
- Return event location information
- Return event geometry
- Return GDACS event URLs
- Return map image URLs
- Return event metadata
- Handle GDACS API errors

### Use Cases

- Monitor global disaster events
- Monitor earthquakes
- Monitor floods
- Monitor tropical cyclones
- Build emergency alert workflows
- Send disaster notifications
- Create disaster monitoring dashboards
- Filter high-priority disaster alerts
- Store disaster events in a database
- Trigger workflows based on disaster severity
- Build humanitarian and emergency response workflows

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `eventlist` | `string` | ❌ No | `TC;FL;EQ` | List of GDACS event types to retrieve. |
| `alertlevel` | `string` | ❌ No | `green;orange;red` | Alert levels to retrieve. |
| `limit` | `number` | ❌ No | `100` | Maximum number of events to return. |

---

## Event Types

The `eventlist` parameter determines which disaster event types are requested.

The default value is:

```text
TC;FL;EQ
```

The supported event type identifiers used by this node are:

| Code | Event Type |
| ---- | ---------- |
| `TC` | Tropical Cyclone |
| `FL` | Flood |
| `EQ` | Earthquake |

Multiple event types can be separated using semicolons.

### Example

```json
{
  "eventlist": "TC;FL;EQ"
}
```

### Tropical Cyclones Only

```json
{
  "eventlist": "TC"
}
```

### Floods Only

```json
{
  "eventlist": "FL"
}
```

### Earthquakes Only

```json
{
  "eventlist": "EQ"
}
```

### Multiple Event Types

```json
{
  "eventlist": "TC;EQ"
}
```

---

## Alert Levels

The `alertlevel` parameter determines which GDACS alert levels are requested.

The default value is:

```text
green;orange;red
```

The supported alert levels are:

| Level | Description |
| ----- | ----------- |
| `green` | Green alert level. |
| `orange` | Orange alert level. |
| `red` | Red alert level. |

Multiple alert levels can be separated using semicolons.

### All Alert Levels

```json
{
  "alertlevel": "green;orange;red"
}
```

### Red Alerts Only

```json
{
  "alertlevel": "red"
}
```

### Orange and Red Alerts

```json
{
  "alertlevel": "orange;red"
}
```

---

## Limit

The `limit` parameter controls the maximum number of events returned by the node.

The default value is:

```json
{
  "limit": 100
}
```

For example:

```json
{
  "limit": 10
}
```

returns a maximum of 10 events.

If GDACS returns fewer events than the configured limit, all available events are returned.

---

## API Endpoint

The node uses the following GDACS API endpoint:

```text
https://www.gdacs.org/gdacsapi/api/events/geteventlist/SEARCH
```

The `eventlist` and `alertlevel` values are sent as query parameters.

Example request:

```text
https://www.gdacs.org/gdacsapi/api/events/geteventlist/SEARCH?eventlist=TC%3BFL%3BEQ&alertlevel=green%3Borange%3Bred
```

The API endpoint is defined internally by the node and is not exposed as a configuration parameter.

---

## Inputs

This node does not require workflow input.

All configuration is provided through the node configuration.

Example:

```json
{
  "eventlist": "TC;FL;EQ",
  "alertlevel": "green;orange;red",
  "limit": 100
}
```

The workflow input received by `handleTick()` is not used by the current implementation.

---

## Outputs

The node returns a structured object containing the query information and disaster events.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Indicates that the GDACS request succeeded. |
| `query` | `object` | Contains the event filters and configured limit. |
| `events` | `array` | List of disaster events returned by GDACS. |
| `total_events` | `number` | Total number of events received from the API before applying the limit. |
| `returned_events` | `number` | Number of events returned by the node. |

### Query Fields

The `query` object contains:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `eventlist` | `string` | Event types requested. |
| `alertlevel` | `string` | Alert levels requested. |
| `limit` | `number` | Maximum number of events returned. |

### Output Example

```json
{
  "success": true,
  "query": {
    "eventlist": "TC;FL;EQ",
    "alertlevel": "green;orange;red",
    "limit": 100
  },
  "events": [],
  "total_events": 0,
  "returned_events": 0
}
```

The exact event data depends on the current response from the GDACS API.

---

## Event Fields

Each returned event may contain the following fields:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `eventid` | `string` | GDACS event identifier. |
| `eventname` | `string` | Name of the disaster event. |
| `eventtype` | `string` | Full event type. |
| `eventtypeShort` | `string` | Short event type identifier. |
| `alertlevel` | `string` | GDACS alert level. |
| `alertscore` | `number` | GDACS alert score. |
| `severity` | `number` | Event severity. |
| `severityunit` | `string` | Unit used for the severity value. |
| `country` | `string` | Affected country. |
| `countrycode` | `string` | Country code. |
| `iso3` | `string` | ISO 3-letter country code. |
| `fromdate` | `string` | Event start date. |
| `todate` | `string` | Event end date. |
| `durationinday` | `number` | Event duration in days. |
| `population` | `number` | Population associated with the event. |
| `vulnerability` | `number` | Vulnerability value associated with the event. |
| `version` | `number` | GDACS event version. |
| `geometry` | `object` | Geographic geometry associated with the event. |
| `location` | `string` | Event location. |
| `glide` | `string` | GLIDE identifier, when available. |
| `url` | `string` | GDACS event URL. |
| `mapimageurl` | `string` | URL of the event map image. |
| `publicid` | `string` | Public event identifier. |
| `feed` | `string` | GDACS feed information. |
| `resource` | `string` | Event resource information. |

The exact values and availability of fields depend on the GDACS API response.

---

## Event Output Example

```json
{
  "eventid": "123456",
  "eventname": "Example Event",
  "eventtype": "Earthquake",
  "eventtypeShort": "EQ",
  "alertlevel": "orange",
  "alertscore": 2.5,
  "severity": 6.2,
  "severityunit": "M",
  "country": "Example Country",
  "countrycode": "XX",
  "iso3": "XXX",
  "fromdate": "2026-08-10T12:00:00Z",
  "todate": "2026-08-10T12:30:00Z",
  "durationinday": 0,
  "population": 100000,
  "vulnerability": 0.5,
  "version": 1,
  "geometry": {},
  "location": "Example Location",
  "glide": "EQ-2026-000000",
  "url": "https://www.gdacs.org/",
  "mapimageurl": "https://www.gdacs.org/",
  "publicid": "public-id",
  "feed": "GDACS",
  "resource": "event-resource"
}
```

The example above is illustrative. Actual event values are provided by GDACS.

---

## Event Filtering

The node sends the configured filters to the GDACS API.

### Example: Red Earthquake Alerts

```json
{
  "eventlist": "EQ",
  "alertlevel": "red",
  "limit": 20
}
```

### Example: Tropical Cyclones

```json
{
  "eventlist": "TC",
  "alertlevel": "green;orange;red",
  "limit": 50
}
```

### Example: Floods

```json
{
  "eventlist": "FL",
  "alertlevel": "orange;red",
  "limit": 25
}
```

---

## Total and Returned Events

The node provides two event count fields.

### `total_events`

`total_events` represents the number of events received from the GDACS API before the configured limit is applied.

Example:

```json
{
  "total_events": 250
}
```

### `returned_events`

`returned_events` represents the number of events actually returned by the node.

Example:

```json
{
  "total_events": 250,
  "returned_events": 100
}
```

If fewer events are available than the configured limit:

```json
{
  "total_events": 25,
  "returned_events": 25
}
```

---

## Default Configuration

If no configuration values are supplied, the node uses:

```json
{
  "eventlist": "TC;FL;EQ",
  "alertlevel": "green;orange;red",
  "limit": 100
}
```

The implementation also applies these fallback values when the corresponding configuration value is empty:

```text
eventlist → TC;FL;EQ
alertlevel → green;orange;red
limit → 100
```

---

## Workflow Integration

### Sample Workflow: Get Disaster Alerts

```json
{
  "nodes": [
    {
      "id": "gdacs",
      "type": "gdacs",
      "config": {
        "eventlist": "TC;FL;EQ",
        "alertlevel": "green;orange;red",
        "limit": 100
      }
    }
  ]
}
```

### Sample Workflow: GDACS → If

```json
{
  "nodes": [
    {
      "id": "gdacs",
      "type": "gdacs",
      "config": {
        "eventlist": "EQ",
        "alertlevel": "red",
        "limit": 20
      }
    },
    {
      "id": "check-alert",
      "type": "if"
    }
  ]
}
```

The `If` node can inspect fields such as:

```text
alertlevel
alertscore
severity
country
population
vulnerability
eventtypeShort
```

### Sample Workflow: GDACS → Function

```json
{
  "nodes": [
    {
      "id": "gdacs",
      "type": "gdacs",
      "config": {
        "eventlist": "EQ",
        "alertlevel": "orange;red",
        "limit": 50
      }
    },
    {
      "id": "process-events",
      "type": "function"
    }
  ]
}
```

The `Function` node can transform or process the returned disaster events.

### Common Patterns

- Schedule → GDACS → If → Notification
- GDACS → Function → Database
- GDACS → If → Email
- GDACS → Function → Dashboard
- GDACS → If → HTTP Request
- GDACS → Notification → Emergency Response

---

## Error Handling

If the GDACS API returns a non-success HTTP status, the node throws an error.

The API error uses the following format:

```text
GDACS API error: <status>
```

The node then wraps the error as:

```text
GDACS query failed: <error message>
```

### Example

If the GDACS API returns HTTP status `500`:

```text
GDACS query failed: GDACS API error: 500
```

---

## Troubleshooting

### GDACS API Request Failed

**Cause**

The GDACS API returned a non-success HTTP status.

**Solution**

Verify that the GDACS API is available and check the HTTP status returned by the service.

---

### No Events Returned

**Cause**

The selected event types and alert levels may not currently have matching events.

**Solution**

Try broader filters:

```json
{
  "eventlist": "TC;FL;EQ",
  "alertlevel": "green;orange;red",
  "limit": 100
}
```

---

### Too Many Events

**Cause**

The GDACS API returned more events than the workflow requires.

**Solution**

Reduce the `limit` value:

```json
{
  "limit": 10
}
```

---

### High-Priority Events Only

To request only red alerts:

```json
{
  "eventlist": "TC;FL;EQ",
  "alertlevel": "red",
  "limit": 100
}
```

---

## API Response Handling

The node supports multiple possible response structures from the GDACS API.

If the API response is an array, the node uses that array as the event list.

If the API response is an object, the node checks:

1. `features`
2. `results`

If neither property is available, the node uses an empty event array.

The resulting events are then limited using the configured `limit`.

### Processing Logic

```text
GDACS API Response
        ↓
Is response an array?
        ↓
      Yes
        ↓
   Use response
        │
        └───────────────┐
                        │
       No               │
        ↓               │
Check "features"        │
        ↓               │
Check "results"         │
        ↓               │
No matching data        │
        ↓               │
   Empty array ─────────┘
        ↓
Apply limit
        ↓
Map event fields
        ↓
Return structured output
```

---

## Data Transformation

The node maps each returned event to the following fields:

```text
eventid
eventname
eventtype
eventtypeShort
alertlevel
alertscore
severity
severityunit
country
countrycode
iso3
fromdate
todate
durationinday
population
vulnerability
version
geometry
location
glide
url
mapimageurl
publicid
feed
resource
```

This provides a consistent structure for downstream workflow nodes.

---

## Implementation Behavior

The node performs the following steps:

1. Reads `eventlist`, `alertlevel`, and `limit` from the configuration.
2. Applies fallback values when configuration values are empty.
3. Creates URL query parameters.
4. Sends a `GET` request to the GDACS API.
5. Checks the HTTP response status.
6. Parses the JSON response.
7. Detects the returned event array.
8. Applies the configured limit.
9. Maps the supported event fields.
10. Returns structured disaster event data.

---

## Processing Flow

```text
Node Configuration
        ↓
Event Type Filter
        ↓
Alert Level Filter
        ↓
GDACS API Request
        ↓
HTTP Response
        ↓
Parse JSON
        ↓
Extract Events
        ↓
Apply Limit
        ↓
Map Event Fields
        ↓
Return Disaster Alerts
```

---

## Privacy Considerations

The node retrieves disaster information from the GDACS API.

The node does not require personal information or user-specific credentials.

Workflow builders should consider the sensitivity of any downstream processing when storing or distributing disaster and location information.

---

## Related

- [Function](./function.md) – Transform and process disaster events
- [If](./if.md) – Filter and route disaster events
- [HTTP Request](./http-request.md) – Make requests to external APIs
- [Notification](./notification.md) – Send disaster alerts to users

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-10 | Initial release |

---
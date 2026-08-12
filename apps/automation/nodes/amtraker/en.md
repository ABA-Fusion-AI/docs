---
node_id: "amtraker"
title: "Amtraker"
description: "Get Amtrak train information from the Amtraker API."
category: "Transportation"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:

- amtrak
- amtraker
- trains
- transportation
- transit
- rail
- api
- real-time
- tracking
- usa

related_nodes:
- function
- if
- http-request

---

# Amtraker

> **Category:** transportation-nodes | **Type:** Action Node

Fetch live **Amtrak train** information from the **Amtraker** API.

The **Amtraker** node calls the public `api-v3.amtraker.com` service and returns the full list of currently tracked Amtrak trains as structured workflow data.

### Supported Features

- Fetch all currently tracked Amtrak trains
- No configuration parameters required
- HTTP error handling with status code reporting
- Wrapped success/data response shape
- Support workflow-based Amtrak monitoring

### Use Cases

- Monitor live Amtrak train positions and statuses
- Build a train-tracking dashboard
- Notify users when a specific train is delayed or arriving
- Filter trains by route, station, or status using an `If` node
- Transform or reduce the train list using a `Function` node
- Feed train data into a map or visualization pipeline
- Log train snapshots to a database over time

---

## Configuration

### Parameters

This node has **no configuration parameters**. It takes no input and requires no setup.

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| — | — | — | — | This node has no configurable parameters. |

---

## API Details

The node calls the following endpoint:

```text
GET https://api-v3.amtraker.com/v3/trains
```

No API key or authentication is required.

The full response body is returned as-is under the `data` field, with no filtering, parsing, or transformation applied.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input.

Incoming workflow data is not used to filter or select trains — the node always fetches the full trains list.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `data` | `object` | Raw JSON response from the Amtraker API, keyed by train number. |

The node does not define a fixed schema for `data` — its shape depends entirely on what the Amtraker API currently returns (train number, route name, current location, status, stations, timings, etc.).

---

## Output Example

```json
{
  "success": true,
  "data": {
    "1": [
      {
        "routeName": "Amtrak Cascades",
        "trainNum": "1",
        "trainID": "1-2026-08-12",
        "lat": 45.5231,
        "lon": -122.6765,
        "velocity": 79,
        "heading": "S",
        "statusMsg": "Route Origin",
        "origTimestamp": "2026-08-12 07:00:00",
        "destTimestamp": "2026-08-12 21:30:00",
        "stations": []
      }
    ]
  }
}
```

The exact shape, fields, and number of trains depend entirely on the live Amtraker API response at request time.

---

## Configuration Examples

### Default Configuration

The node has no parameters, so it is always used with an empty configuration.

```json
{}
```

---

## Workflow Integration

### Sample Workflow: Fetch All Trains

```json
{
  "nodes": [
    {
      "id": "amtraker",
      "type": "amtraker",
      "config": {}
    }
  ]
}
```

### Sample Workflow: Amtraker → Function

```json
{
  "nodes": [
    {
      "id": "amtraker",
      "type": "amtraker",
      "config": {}
    },
    {
      "id": "process-trains",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Amtraker → If

```json
{
  "nodes": [
    {
      "id": "amtraker",
      "type": "amtraker",
      "config": {}
    },
    {
      "id": "filter-delayed-trains",
      "type": "if"
    }
  ]
}
```

### Common Patterns

- Schedule → Amtraker → Function (filter by route/train number)
- Amtraker → If → Notification (delay alerts)
- Amtraker → Database → Store train snapshots
- Amtraker → Function → Map/visualization pipeline

---

## Error Handling

If the Amtraker API request fails or returns a non-success HTTP status, the node throws an error.

The error format is:

```text
Amtraker request failed: <error message>
```

### API Error Response

If the API responds with a non-OK HTTP status:

```text
Amtraker API error: <status>
```

This inner error is then wrapped into the outer `Amtraker request failed: ...` message.

---

## Troubleshooting

### "Amtraker API error: <status>"

**Cause**

The Amtraker API returned a non-success HTTP status code (e.g. `500`, `503`).

**Solution**

Retry the request later. The Amtraker service is a third-party public API and may experience downtime independent of this node.

---

### "Amtraker request failed: ..."

**Cause**

The underlying fetch call failed entirely (network error, DNS failure, timeout) or the API error above was thrown and wrapped.

**Solution**

Check network connectivity to `api-v3.amtraker.com` and verify the service status before retrying.

---

### Empty or Unexpected `data`

**Cause**

The Amtraker API response shape can vary depending on which trains are currently active; not all fields are guaranteed to be present for every train.

**Solution**

Use a `Function` node downstream to defensively access nested fields and handle missing data.

---

### Large Response Size

**Cause**

The `/v3/trains` endpoint returns **all** currently tracked trains with no filtering option, which can be a large payload during peak hours.

**Solution**

Use a `Function` node after Amtraker to filter down to the specific train numbers or routes needed before further processing.

---

## Security

The node performs outbound HTTP requests to the public Amtraker API (`api-v3.amtraker.com`).

No API key or authentication credential is required or accepted by the node.

The node does not accept a user-provided URL — the endpoint is fixed internally, preventing it from being used as a generic arbitrary URL fetcher.

---

## Notes

The node returns the raw Amtraker API response rather than a normalized or filtered structure.

The node does not:

- Filter trains by route, station, or number
- Normalize or validate the response schema
- Cache results between calls
- Store train data
- Generate notifications or alerts
- Compute delays or ETAs itself

It is intended to retrieve the current Amtrak trains snapshot for downstream workflow processing.

---

## Related

- [Function](./function.md) – Filter, transform, and process train data
- [If](./if.md) – Route workflows based on train status or delay conditions
- [HTTP Request](./http-request.md) – Make additional Amtraker API calls (e.g. per-station or per-train endpoints)
---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-12 | Initial release |
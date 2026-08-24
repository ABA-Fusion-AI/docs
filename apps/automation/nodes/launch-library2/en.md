---
node_id: "launch-library-2"
title: "Launch Library 2"
description: "Get space launch data from Launch Library 2 API."
category: "Space / Aerospace"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- space
- launches
- rockets
- aerospace
- thespacedevs
- launch-library
- api

related_nodes:
- function
- if
- http-request

---

# Launch Library 2

> **Category:** space-nodes | **Type:** Action Node

Get **space launch data** — upcoming and past rocket launches — from **Launch Library 2**, The Space Devs' public spaceflight data API.

The **Launch Library 2** node queries the `/launch/` endpoint with an optional search term, response detail mode, and pagination, and returns the launch list alongside pagination metadata.

### Supported Features

- Free-text search across launches
- Selectable response detail mode (`list` or `detailed`)
- Pagination via `limit`/`offset`
- Pass-through of the API's `next`/`previous` pagination links
- Simple, direct pass-through of the raw launch objects (no field reshaping)

### Use Cases

- Display upcoming rocket launches on a dashboard
- Search for launches by mission, rocket, or agency name
- Build a launch-day notification system
- Track a specific launch provider's schedule (e.g. search by agency name)
- Feed launch data into a `Function` node for calendar or countdown widgets

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `search` | `string` | ❌ No | `""` | Free-text search term. Only sent to the API if non-empty. |
| `mode` | `string` | ❌ No | `"list"` | Response detail mode: `list` (summary fields) or `detailed` (full launch data). |
| `limit` | `number` | ❌ No | `10` | Number of results to return per page. |
| `offset` | `number` | ❌ No | `0` | Number of results to skip, for pagination. |

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `query` | `object` | Echo of the query used: `{ search, mode, limit, offset }` (`search` is `undefined` if empty). |
| `total_results` | `number` | Total number of matching launches, from the API's `count` field (falls back to `results.length` if `count` is missing). |
| `launches` | `array` | Raw array of launch objects, as returned by the Launch Library 2 API — not reshaped by this node. |
| `pagination.next` | `string \| null` | URL for the next page of results, or `null` if there isn't one. |
| `pagination.previous` | `string \| null` | URL for the previous page of results, or `null` if there isn't one. |
| `note` | `string` | Always `"Returns space launch data. Mode: list or detailed"`. |

The shape of each object in `launches` depends entirely on the selected `mode` — `detailed` returns significantly more fields (full rocket, pad, agency, and mission details) than `list`.

---

## Output Example

### `list` Mode (abbreviated)

```json
{
  "success": true,
  "query": {
    "search": undefined,
    "mode": "list",
    "limit": 10,
    "offset": 0
  },
  "total_results": 187,
  "launches": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "name": "Falcon 9 Block 5 | Starlink Group 12-5",
      "status": { "id": 1, "name": "Go" },
      "net": "2026-08-26T14:32:00Z",
      "launch_service_provider": { "id": 121, "name": "SpaceX" },
      "rocket": { "id": 5087 },
      "pad": { "id": 21, "name": "SLC-40" }
    }
  ],
  "pagination": {
    "next": "https://ll.thespacedevs.com/2.2.0/launch/?limit=10&offset=10",
    "previous": null
  },
  "note": "Returns space launch data. Mode: list or detailed"
}
```

### No Results

```json
{
  "success": true,
  "query": { "search": "nonexistent mission xyz", "mode": "list", "limit": 10, "offset": 0 },
  "total_results": 0,
  "launches": [],
  "pagination": { "next": null, "previous": null },
  "note": "Returns space launch data. Mode: list or detailed"
}
```

---

## Configuration Examples

### Default (Upcoming Launches, List Mode)

```json
{
  "mode": "list",
  "limit": 10
}
```

### Search by Mission or Provider

```json
{
  "search": "SpaceX",
  "mode": "list",
  "limit": 20
}
```

### Detailed Mode

```json
{
  "mode": "detailed",
  "limit": 5
}
```

### Paginated (Second Page)

```json
{
  "mode": "list",
  "limit": 10,
  "offset": 10
}
```

---

## Workflow Integration

### Sample Workflow: Fetch Launches → Function

```json
{
  "nodes": [
    {
      "id": "launch-library",
      "type": "launch-library-2",
      "config": {
        "mode": "list",
        "limit": 5
      }
    },
    {
      "id": "build-dashboard",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Search → If → Notification

```json
{
  "nodes": [
    {
      "id": "launch-library-search",
      "type": "launch-library-2",
      "config": {
        "search": "Starship",
        "mode": "list"
      }
    },
    {
      "id": "check-upcoming",
      "type": "if"
    },
    {
      "id": "notify-launch-day",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Schedule → Launch Library → Database

```json
{
  "nodes": [
    {
      "id": "launch-library",
      "type": "launch-library-2",
      "config": {
        "mode": "detailed",
        "limit": 20
      }
    },
    {
      "id": "log-launch-schedule",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- Schedule (daily) → Launch Library 2 → If (launch within 24h) → Notification — launch-day alerts
- Launch Library 2 → Function → Calendar/widget rendering
- Launch Library 2 (`detailed`) → Database → dashboard/reporting pipeline

---

## Error Handling

### API Error

```text
Launch Library 2 API error: <status>
```

Raised when the API returns a non-OK HTTP status — including `429` if the API's rate limit is exceeded.

### Wrapped Failure

```text
Launch Library 2 lookup failed: <underlying error message>
```

All errors, including the API error above, are re-thrown wrapped in this message from `handleTick`.

---

## Troubleshooting

### "Launch Library 2 lookup failed: Launch Library 2 API error: 429"

**Cause**

Launch Library 2's public API enforces a fairly strict rate limit on its free tier (commonly a handful of requests per hour without an API key/subscription).

**Solution**

Reduce the frequency of calls to this node, cache results downstream, or use a Space Devs paid tier if higher throughput is needed.

---

### "Launch Library 2 lookup failed: Launch Library 2 API error: <status>"

**Cause**

A malformed request — for example, an invalid `mode` value not recognized by the API.

**Solution**

Confirm `mode` is set to either `"list"` or `"detailed"`.

---

### `launches` Fields Vary Between Calls

**Cause**

The `mode` parameter significantly changes the shape of each launch object — switching between `list` and `detailed` will change which fields are present without any error.

**Solution**

Design downstream `Function` nodes defensively, or standardize on a single `mode` per workflow.

---

### Missing Launches You Expect to See

**Cause**

`search` performs a free-text match and may not catch every relevant launch (e.g. searching `"SpaceX"` may miss launches where the provider name is stored differently); alternatively, `limit`/`offset` may be excluding the launch from the current page.

**Solution**

Try a broader search term, or paginate through additional pages using `offset` and the returned `pagination.next` URL.

---

## Security

The node performs outbound HTTP requests to the public Launch Library 2 API (`ll.thespacedevs.com`).

No API key or authentication credential is required for this node's usage of the free public endpoint.

---

## Notes

The node returns the raw launch objects from the API with no reshaping — the available fields depend entirely on the selected `mode`.

The node does not:

- Send an API key (only the free, rate-limited tier is used)
- Support endpoints other than `/launch/` (e.g. `/agencies/`, `/astronaut/`, `/event/` are not covered)
- Cache results between calls
- Automatically paginate through all results in a single call
- Retry on rate-limit (429) errors

It is intended to provide straightforward, searchable access to launch schedule data for downstream space-tracking and notification workflows.


---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
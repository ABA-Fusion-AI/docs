---
node_id: "space-x-launches"
title: "SpaceX Launch Library"
description: "Get upcoming or previous SpaceX launches from the Launch Library 2 API (The Space Devs)."
category: "Space / Aerospace"
subcategory: "Launch Data"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - spacex
  - space
  - launches
  - rockets
  - thespacedevs
  - launch-library
related_nodes:
  - launch-library2
  - function
  - filter
  - http-request
---

<!-- SECTION: overview -->
# SpaceX Launch Library

> **Category:** Space / Aerospace &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Retrieve **SpaceX** launch data from the public **Launch Library 2 API** by The Space Devs (`https://ll.thespacedevs.com/2.3.0`). The node filters launches on `SpaceX` and supports two endpoints:
- **`upcoming`:** Next scheduled SpaceX launches.
- **`previous`:** Most recent past SpaceX launches.

No API key is required.

### Use Cases

- **Launch alerts:** Notify a channel or a mailing list before the next SpaceX launch.
- **Dashboards:** Feed a space-themed dashboard with upcoming launch dates, rockets and launch pads.
- **Mission history:** Collect recent launches and their status (success, failure) for reporting.
- **Content automation:** Generate daily posts or newsletters about upcoming missions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `endpoint` | `enum` | No | `upcoming` | Launches to retrieve: `upcoming` or `previous`. |
| `limit` | `number` | No | `10` | Maximum number of launches to return. Clamped between `1` and `100`. |

### Endpoints

| Endpoint | API Route | Returns |
|----------|-----------|---------|
| `upcoming` | `GET /launches/upcoming/?search=SpaceX` | Future SpaceX launches, closest first |
| `previous` | `GET /launches/previous/?search=SpaceX` | Past SpaceX launches, most recent first |

### Authentication

No authentication is needed. The Launch Library 2 free tier is rate-limited (around 15 requests per hour per IP), so avoid running this node in tight loops or on very frequent schedules.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Launch list with pagination information (see below). |
| `error` | `Error` | Emitted when the API call fails. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Always `true` on a successful call. |
| `endpoint` | `string` | Endpoint used (`upcoming` or `previous`). |
| `total_results` | `number` | Total number of matching launches in the API. |
| `returned_results` | `number` | Number of launches returned in this response. |
| `next` | `string \| null` | URL of the next page of results. |
| `previous` | `string \| null` | URL of the previous page of results. |
| `launches` | `array` | Launch objects as returned by Launch Library 2. |

### Output Example

```json
{
  "success": true,
  "endpoint": "upcoming",
  "total_results": 42,
  "returned_results": 2,
  "next": "https://ll.thespacedevs.com/2.3.0/launches/upcoming/?format=json&limit=2&offset=2&search=SpaceX",
  "previous": null,
  "launches": [
    {
      "id": "e3df2ecd-c239-472f-95e4-2b89b4f75800",
      "name": "Falcon 9 Block 5 | Starlink Group 10-12",
      "net": "2026-11-20T03:15:00Z",
      "status": { "id": 1, "name": "Go for Launch", "abbrev": "Go" },
      "launch_service_provider": { "id": 121, "name": "SpaceX" },
      "pad": {
        "name": "Space Launch Complex 40",
        "location": { "name": "Cape Canaveral SFS, FL, USA" }
      }
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Get the next SpaceX launches
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **SpaceX Launch Library Node:** Fetches the upcoming SpaceX launches from Launch Library 2.
3. **Log Node:** Displays the launch list.

### Example Configuration

```json
{
  "endpoint": "upcoming",
  "limit": 5
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `SpaceX launches lookup failed: Launch Library 2 API error: 429 Too Many Requests`
- **Cause:** The free-tier rate limit of Launch Library 2 has been reached.
- **Solution:** Wait before running the workflow again, or reduce how often it runs.

#### `SpaceX launches lookup failed: Launch Library 2 API error: 5xx`
- **Cause:** The Launch Library 2 service is temporarily unavailable.
- **Solution:** Retry later.

#### Empty `launches` array
- **Cause:** No SpaceX launch matches the selected endpoint at that moment.
- **Solution:** Try the other endpoint or increase `limit`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: endpoints, parameter reference, output schema, and workflow integration |

<!-- /SECTION: changelog -->

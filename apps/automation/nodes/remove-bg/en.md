---
node_id: "track123"
title: "Track123"
description: "Track packages using Track123 API. Supports 1,700+ carriers including Morocco Post."
category: "Logistics / Shipping"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:

- track123
- package-tracking
- shipping
- logistics
- carriers
- multi-carrier
- api

related_nodes:
- poste-ma
- function
- if
- http-request

---

# Track123

> **Category:** logistics-nodes | **Type:** Action Node

Register and query package tracking numbers via the **Track123** API, a multi-carrier tracking aggregator supporting 1,700+ carriers, including Morocco Post.

The **Track123** node exposes two operations: `Register Tracking`, which submits one or more tracking numbers to Track123 for ongoing monitoring, and `Get Tracking`, which queries the current status of previously registered (or matching) shipments with flexible filters and cursor-based pagination.

### Supported Features

- Register one or more tracking numbers in a single call, with rich optional per-item metadata
- Query tracking status by tracking numbers, order numbers, or a creation-time date range
- Cursor-based pagination for large result sets
- Detailed error messages that surface Track123's own error text and per-item rejection reasons
- Broad carrier coverage (1,700+ carriers) via a single unified API, including Morocco Post

### Use Cases

- Register new shipments for tracking as soon as an order ships
- Poll shipment status across many carriers without integrating each carrier's API individually
- Build a customer-facing order tracking dashboard backed by multiple carriers
- Alert a team when a shipment's status changes (e.g. delivered, exception)
- Reconcile shipment status against order records using `orderNos`

---

## Configuration

### Base Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ✅ Yes | — | Track123 API key, sent as the `Track123-Api-Secret` header. |
| `operation` | `enum` | ❌ No | `"Get Tracking"` | Operation: `Register Tracking` or `Get Tracking`. |

### Register Tracking Parameters

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `trackings` | `array` | ✅ Yes (at least one) | Array of tracking objects to register (fields below). |

Each entry in `trackings`:

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `trackNo` | `string` | ✅ Yes | The tracking number. |
| `courierCode` | `string` | ❌ No | Carrier code, if known (improves matching accuracy). |
| `orderNo` | `string` | ❌ No | Associated order number. |
| `country` | `string` | ❌ No | Destination country. |
| `shipTime` | `string` | ❌ No | Shipment timestamp. |
| `customerEmail` | `string` | ❌ No | Customer email, for notification purposes on Track123's side. |
| `postalCode` | `string` | ❌ No | Destination postal code. |
| `extendFieldMap` | `any` | ❌ No | Arbitrary extra metadata object. |
| `remark` | `string` | ❌ No | Free-text note. |
| `custom1` / `custom2` | `string` | ❌ No | Custom fields for caller-defined metadata. |

### Get Tracking Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `trackNos` | `string[]` | ❌ No | — | Filter by specific tracking numbers. |
| `orderNos` | `string[]` | ❌ No | — | Filter by specific order numbers. |
| `createTimeStart` | `string` | ❌ No | — | Filter to trackings created on/after this timestamp. |
| `createTimeEnd` | `string` | ❌ No | — | Filter to trackings created before this timestamp. |
| `cursor` | `string` | ❌ No | — | Pagination cursor from a previous response, for fetching the next page. |
| `queryPageSize` | `number` | ❌ No | `100` | Results per page, 1–100. |

If none of `trackNos`, `orderNos`, `createTimeStart`, `createTimeEnd`, or `cursor` are set, the request body is effectively empty aside from `queryPageSize` — behavior in that case depends on Track123's API (likely returns the most recent trackings up to the page size).

---

## Operations

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| `Register Tracking` | `POST /gateway/open-api/tk/v2/track/import` | `POST` | Submit tracking numbers for Track123 to begin monitoring. |
| `Get Tracking` | `POST /gateway/open-api/tk/v2/track/query` | `POST` | Query tracking status with optional filters and pagination. |

Note that `Get Tracking` uses `POST` (with filters in the request body), not `GET` with query parameters.

---

## Request Body Construction

### Register Tracking

The `trackings` array is transformed into the API's expected shape, with every optional field explicitly defaulted:

```text
courierCode, orderNo, country, shipTime, customerEmail, postalCode, remark, custom1, custom2 → "" if not provided
extendFieldMap → {} if not provided
```

Unlike `Get Tracking`, **all fields are always sent** for each tracking entry, even if empty — there is no conditional omission.

### Get Tracking

The request body is built up **conditionally** — only fields that are actually set are added:

```text
if trackNos non-empty → include trackNos
if orderNos non-empty → include orderNos
if createTimeStart set → include createTimeStart
if createTimeEnd set → include createTimeEnd
if cursor set → include cursor
if queryPageSize !== undefined → include queryPageSize
```

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `operation` | `string` | The operation that was performed. |
| `data` | `any` | Track123's raw response body for the operation. |
| `statusCode` | `number` | HTTP status code of the response. |

The shape of `data` differs between operations — for `Register Tracking`, it typically includes accepted/rejected tracking entries; for `Get Tracking`, it includes the matched tracking records and pagination info (e.g. a `cursor` for the next page).

---

## Output Example

### `Register Tracking`

```json
{
  "success": true,
  "operation": "Register Tracking",
  "data": {
    "code": "00000",
    "msg": "success",
    "data": {
      "accepted": [
        { "trackNo": "RR123456789MA", "courierCode": "" }
      ],
      "rejected": []
    }
  },
  "statusCode": 200
}
```

### `Get Tracking`

```json
{
  "success": true,
  "operation": "Get Tracking",
  "data": {
    "code": "00000",
    "msg": "success",
    "data": {
      "content": [
        {
          "trackNo": "RR123456789MA",
          "courierCode": "morocco-post",
          "trackStatus": "Delivered",
          "orderNo": "ORD-1001"
        }
      ],
      "cursor": "eyJvZmZzZXQiOjEwMH0="
    }
  },
  "statusCode": 200
}
```

---

## Configuration Examples

### Register a Single Tracking

```json
{
  "apiKey": "your-track123-api-key",
  "operation": "Register Tracking",
  "trackings": [
    {
      "trackNo": "RR123456789MA",
      "orderNo": "ORD-1001",
      "customerEmail": "customer@example.com"
    }
  ]
}
```

### Register Multiple Trackings

```json
{
  "apiKey": "your-track123-api-key",
  "operation": "Register Tracking",
  "trackings": [
    { "trackNo": "RR123456789MA" },
    { "trackNo": "1Z999AA10123456784", "courierCode": "ups" }
  ]
}
```

### Query by Tracking Numbers

```json
{
  "apiKey": "your-track123-api-key",
  "operation": "Get Tracking",
  "trackNos": ["RR123456789MA"]
}
```

### Query by Order Numbers, Paginated

```json
{
  "apiKey": "your-track123-api-key",
  "operation": "Get Tracking",
  "orderNos": ["ORD-1001", "ORD-1002"],
  "queryPageSize": 50
}
```

### Query by Date Range with Cursor

```json
{
  "apiKey": "your-track123-api-key",
  "operation": "Get Tracking",
  "createTimeStart": "2026-08-01T00:00:00Z",
  "createTimeEnd": "2026-09-01T00:00:00Z",
  "cursor": "eyJvZmZzZXQiOjEwMH0="
}
```

---

## Workflow Integration

### Sample Workflow: Order Created → Register Tracking

```json
{
  "nodes": [
    {
      "id": "order-shipped",
      "type": "function"
    },
    {
      "id": "track123-register",
      "type": "track123",
      "config": {
        "apiKey": "your-track123-api-key",
        "operation": "Register Tracking"
      }
    }
  ]
}
```

### Sample Workflow: Query Status → If → Notification

```json
{
  "nodes": [
    {
      "id": "track123-query",
      "type": "track123",
      "config": {
        "apiKey": "your-track123-api-key",
        "operation": "Get Tracking",
        "orderNos": ["ORD-1001"]
      }
    },
    {
      "id": "check-delivered",
      "type": "if"
    },
    {
      "id": "notify-customer",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Schedule → Query All → Database

```json
{
  "nodes": [
    {
      "id": "track123-query-all",
      "type": "track123",
      "config": {
        "apiKey": "your-track123-api-key",
        "operation": "Get Tracking",
        "createTimeStart": "2026-08-01T00:00:00Z"
      }
    },
    {
      "id": "log-tracking-history",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- Order fulfillment → Track123 (`Register Tracking`) — start monitoring as soon as a shipment goes out
- Schedule (hourly) → Track123 (`Get Tracking`) → If (status changed) → Notification — delivery status monitoring
- Track123 (`Get Tracking`, paginated via `cursor`) → Function (loop) → Database — bulk sync of tracking history

---

## Error Handling

### Missing API Key

```text
Track123 API key is required. Please provide it in the node parameters or use a template expression like {{secrets.track123ApiKey}}
```

Raised when `apiKey` is empty or whitespace-only.

### Missing Trackings

```text
At least one tracking is required for Register Tracking operation
```

Raised for `Register Tracking` when `trackings` is empty or unset.

### Unknown Operation

```text
Unknown operation: <operation>
```

### HTTP/API Error

```text
Track123 API error: HTTP error! status: <status> - <errorData.msg>. Rejected: <per-item rejection reasons, if any>
```

Raised when the response is non-OK. The message is built up progressively:
- Base: `HTTP error! status: <status>`
- `+ " - <msg>"` if the error body has a `msg` field.
- `+ ". Rejected: <comma-joined reasons>"` if the error body has a `data.rejected` array — each entry's `error.msg` (or `error.code`, or `"Unknown error"`) is joined into the message.

All of the above is wrapped in the outer `Track123 API error: ...` prefix.

---

## Troubleshooting

### "Track123 API error: Track123 API key is required. ..."

**Cause**

`apiKey` was left empty.

**Solution**

Set a valid Track123 API key, ideally via a secrets/template expression like `{{secrets.track123ApiKey}}`.

---

### "Track123 API error: At least one tracking is required for Register Tracking operation"

**Cause**

`trackings` is empty while `operation` is `Register Tracking`.

**Solution**

Provide at least one object with a `trackNo` in `trackings`.

---

### "Track123 API error: HTTP error! status: 200 - ... Rejected: ..."

**Cause**

Some or all submitted trackings were rejected by Track123 during registration — common reasons include an unrecognized `trackNo` format, a duplicate registration, or an unsupported/ambiguous carrier when `courierCode` isn't specified.

**Solution**

Check the per-item rejection reasons in the error message; providing `courierCode` explicitly for ambiguous tracking number formats often resolves matching issues.

---

### "Track123 API error: HTTP error! status: 401" or Similar Auth Error

**Cause**

The `apiKey` is invalid, revoked, or doesn't have access to the requested operation.

**Solution**

Verify the API key in the Track123 dashboard.

---

### `Get Tracking` Returns No Results Despite Registered Trackings

**Cause**

The query filters (`trackNos`, `orderNos`, `createTimeStart`/`createTimeEnd`) may not match any registered tracking, or the trackings were registered too recently for Track123 to have processed/indexed them yet.

**Solution**

Double-check the filter values against what was actually submitted in `Register Tracking`, and allow some time after registration before querying.

---

### Only Some Results Returned; More Data Exists

**Cause**

`Get Tracking` is paginated — a single call returns at most `queryPageSize` (default 100) results, with a `cursor` in the response for continuing to the next page.

**Solution**

Pass the returned `cursor` value into a subsequent `Get Tracking` call (typically via a `Function`-driven loop) to retrieve additional pages.

---

## Security

The node performs outbound HTTP requests to Track123's API (`api.track123.com`).

`apiKey` is sent as a custom header (`Track123-Api-Secret`), not as a URL query parameter or in the request body.

`Register Tracking` accepts a `customerEmail` field per tracking — be mindful this constitutes customer PII being sent to a third-party tracking aggregator; handle in line with applicable data-privacy requirements.

---

## Notes

The node returns Track123's raw response body under `data` for both operations, with no reshaping.

The node does not:

- Support carrier lookup/detection separately from registration (courier matching happens on Track123's side)
- Automatically paginate through all `Get Tracking` results — `cursor` must be passed manually across calls
- Support updating or deleting previously registered trackings (register/query only)
- Cache results between calls

It is intended to provide unified, multi-carrier package tracking — including Morocco Post — for downstream logistics and order-fulfillment workflows.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-02 | Initial release |
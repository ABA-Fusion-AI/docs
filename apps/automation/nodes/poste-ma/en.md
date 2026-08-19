---
node_id: "poste-ma"
title: "Poste.ma"
description: "Track packages using Morocco Post (Poste.ma) API"
category: "Logistics / Shipping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- poste-ma
- morocco-post
- package-tracking
- shipping
- logistics
- maroc
- barid-al-maghrib
- html-scraping

related_nodes:
- function
- if
- http-request

---

# Poste.ma

> **Category:** logistics-nodes | **Type:** Action Node

Track a package using **Morocco Post (Poste.ma / Barid Al-Maghrib)**'s public web tracking page.

The **Poste.ma** node fetches Poste.ma's HTML tracking results page for a given barcode and parses the embedded results table into structured tracking data, since Poste.ma does not expose a public JSON API.

### Supported Features

- Package tracking by barcode (`codeAbars`)
- HTML table parsing with a primary targeted selector and a generic fallback
- HTML entity decoding (`&nbsp;`, `&amp;`, `&lt;`, `&gt;`, `&quot;`, `&#39;`, `&apos;`)
- Tag stripping to extract plain text from table cells
- Graceful parsing failure (returns an empty result list rather than throwing on unexpected HTML)

### Use Cases

- Track a Poste.ma shipment's status and delivery history
- Build a customer-facing order tracking widget backed by Moroccan postal shipments
- Monitor delivery status changes for a batch of tracked packages
- Alert a team when a package reaches a specific status (e.g. "Delivered", "Out for Delivery")
- Log tracking history to a database for shipment auditing

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"Track Package"` | Operation to perform. Currently only `"Track Package"` is supported. |
| `codeAbars` | `string` | ✅ Yes | — | The package's tracking barcode number. |

---

## How It Works

1. Builds a `GET` request URL to `https://www.poste.ma/wps/PA_rtletwebtrackingV2/listCode.jsp?codeAbars=<barcode>`.
2. Sends the request with browser-like `Accept` and `User-Agent` headers (Poste.ma serves an HTML page, not a JSON API).
3. Retrieves the raw HTML response.
4. Parses the HTML for a table inside `<div id="web_Tracking_Suivi">` with `id="tab"`.
5. If that specific table isn't found, **falls back** to parsing any `<tr>` rows found anywhere in the response.
6. Skips the first row (assumed to be the table header) and parses each subsequent row into a structured `TrackingRow` object.
7. Returns the parsed rows alongside the request metadata.

---

## HTML Parsing Details

Table rows are parsed by extracting all `<td>`/`<th>` cells via regex, stripping HTML tags, and decoding a fixed set of HTML entities.

**A row is only kept if it has at least 7 cells** — rows with fewer cells are discarded (returns `null` from `parseTableRow`).

Cells are mapped positionally to fields, based on the expected Poste.ma table column order:

| Cell Index | Field | Column (French) |
| ---------- | ----- | ---------------- |
| 0 | `codeBarre` | Code à barre |
| 1 | `dateDepot` | Date Dépôt |
| 2 | `lieuDepot` | Lieu Dépôt |
| 3 | `dateStatut` | Date Statut |
| 4 | `destinataire` | Destinataire |
| 5 | `destination` | Destination |
| 6 | `statut` | Statut |
| 7 | `action` | Action |

This mapping is **positional, not label-based** — if Poste.ma changes its table column order or structure, the field mapping will silently become incorrect rather than raising an error.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call (parsing failures do not set this to `false` — see [Notes](#notes)). |
| `operation` | `string` | The operation performed (`"Track Package"`). |
| `codeAbars` | `string` | The barcode that was queried. |
| `data` | `TrackingRow[]` | Array of parsed tracking history rows, oldest/newest order as returned by Poste.ma. Empty array if no rows were found or parsing failed. |
| `statusCode` | `number` | HTTP status code of the Poste.ma response. |

### TrackingRow Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `codeBarre` | `string` | Barcode, as shown in the tracking table. |
| `dateDepot` | `string` | Deposit/shipment date. |
| `lieuDepot` | `string` | Deposit/shipment location. |
| `dateStatut` | `string` | Date of the current status entry. |
| `destinataire` | `string` | Recipient name. |
| `destination` | `string` | Destination location. |
| `statut` | `string` | Status label (e.g. delivered, in transit). |
| `action` | `string` | Action/event description, if present. |

---

## Output Example

```json
{
  "success": true,
  "operation": "Track Package",
  "codeAbars": "RR123456789MA",
  "data": [
    {
      "codeBarre": "RR123456789MA",
      "dateDepot": "12/08/2026",
      "lieuDepot": "CASABLANCA PRINCIPAL",
      "dateStatut": "15/08/2026",
      "destinataire": "JOHN DOE",
      "destination": "RABAT",
      "statut": "Livré",
      "action": "Remis au destinataire"
    }
  ],
  "statusCode": 200
}
```

### No Tracking Data Found

```json
{
  "success": true,
  "operation": "Track Package",
  "codeAbars": "INVALIDCODE",
  "data": [],
  "statusCode": 200
}
```

---

## Configuration Examples

### Track a Package

```json
{
  "operation": "Track Package",
  "codeAbars": "RR123456789MA"
}
```

---

## Workflow Integration

### Sample Workflow: Track → Function

```json
{
  "nodes": [
    {
      "id": "poste-ma-track",
      "type": "poste-ma",
      "config": {
        "operation": "Track Package",
        "codeAbars": "RR123456789MA"
      }
    },
    {
      "id": "format-status",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Track → If → Notification

```json
{
  "nodes": [
    {
      "id": "poste-ma-track",
      "type": "poste-ma",
      "config": {
        "operation": "Track Package",
        "codeAbars": "RR123456789MA"
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

### Sample Workflow: Schedule → Track → Database

```json
{
  "nodes": [
    {
      "id": "poste-ma-track",
      "type": "poste-ma",
      "config": {
        "operation": "Track Package",
        "codeAbars": "RR123456789MA"
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

- Schedule (daily) → Poste.ma → If (status changed) → Notification — delivery status monitoring
- Poste.ma → Database → Chart/visualization pipeline — shipment history tracking
- Function (loop over barcodes) → Poste.ma → Function (aggregate) — batch tracking

---

## Error Handling

### Missing Barcode

```text
Poste.ma API error: Barcode code (codeAbars) is required for Track Package operation
```

Raised when `codeAbars` is empty or whitespace-only.

### Unknown Operation

```text
Poste.ma API error: Unknown operation: <operation>
```

Should not normally occur given the single-value `operation` enum.

### HTTP Error

```text
Poste.ma API error: HTTP error! status: <status>
```

Raised when the Poste.ma page request returns a non-OK HTTP status.

All errors are wrapped in the outer `Poste.ma API error: ...` message.

---

## Troubleshooting

### "Poste.ma API error: Barcode code (codeAbars) is required for Track Package operation"

**Cause**

`codeAbars` was left empty or contains only whitespace.

**Solution**

Provide a valid tracking barcode.

---

### "Poste.ma API error: HTTP error! status: <status>"

**Cause**

Poste.ma's tracking page is temporarily unavailable, or the request was blocked (e.g. due to missing/changed headers Poste.ma expects from a browser).

**Solution**

Retry later; if the issue persists, Poste.ma may have changed its request requirements, which would need an update to the node's headers.

---

### `data` is an Empty Array Despite a Valid Barcode

**Cause**

Either the barcode has no tracking history yet, the barcode is invalid/unrecognized by Poste.ma, or Poste.ma changed its HTML structure so the table (or its `id="web_Tracking_Suivi"` / `id="tab"` selectors) is no longer found — silently falling through to zero results rather than an error.

**Solution**

Verify the barcode manually on the Poste.ma tracking website. If it shows results there but not through this node, the HTML structure may have changed and the node's parsing logic would need to be updated.

---

### Field Values Look Shifted or Wrong (e.g. `statut` contains a date)

**Cause**

The parsing is **strictly positional** (cell index 0 → 7) — if Poste.ma adds, removes, or reorders a table column, every field after that point will be mapped to the wrong value without any error being raised.

**Solution**

Compare a raw fetch of the Poste.ma tracking page against the expected 8-column structure; if it has changed, the node's `parseTableRow` field mapping needs to be updated to match.

---

### Rows Silently Missing

**Cause**

Any row with fewer than 7 parsed cells is discarded entirely (`parseTableRow` returns `null`) — a malformed or partially-rendered row is dropped rather than included with missing fields.

**Solution**

This is expected defensive behavior for malformed HTML; if entire tracking entries are missing, verify directly on Poste.ma's site.

---

## Security

The node performs outbound HTTP requests to Poste.ma's public tracking page (`www.poste.ma`), sending a browser-like `User-Agent` and `Accept` header to mimic a normal browser request.

No API key or authentication credential is required — this is unauthenticated, publicly accessible tracking data tied only to a barcode value.

---

## Notes

Poste.ma does not provide an official public JSON API for tracking — this node **scrapes and parses the HTML tracking results page**, which makes it inherently fragile to upstream markup changes.

`success: true` is returned even when `data` ends up empty due to a parsing failure — errors during row/table parsing are caught internally and logged to the console (`console.error`), not surfaced in the node's output or thrown as an error.

The node does not:

- Support any operation other than `Track Package` (the enum currently has a single value)
- Validate the barcode format before sending the request
- Retry on HTML structure mismatches
- Surface parsing errors in its output (only via server-side console logging)
- Cache tracking results between calls

It is intended to provide best-effort, unofficial access to Poste.ma package tracking for downstream logistics workflows, with the understanding that it depends on the stability of Poste.ma's public HTML page structure.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release |
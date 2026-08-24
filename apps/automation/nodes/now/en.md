---
node_id: "now"
title: "Now"
description: "Get the current UTC date/time in the selected output type."
category: "data-transformation-etl"
subcategory: "date-time"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - now
  - date
  - time
  - timestamp
  - utc
  - iso8601
  - date-time
related_nodes:
  - format-date
  - date-add
  - date-compare
  - convert-timezone
---

<!-- SECTION: header -->
# Now

> **Category:** Data Transformation (ETL) | **Subcategory:** Date & Time | **Type:** Action Node

Get the current UTC date and time as a formatted string or a Unix timestamp in milliseconds.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Now** node retrieves the current UTC date and time at the exact moment of execution. It outputs either a Unix millisecond timestamp (number) or a formatted date string matching common international standards (such as ISO 8601, RFC 2822, YYYY-MM-DD, or DD/MM/YYYY).

Because it computes UTC timestamps directly without external network requests, the node executes instantaneously and provides a reliable time source for all downstream workflow steps.

### Key Features

- **UTC Time Source:** Generates consistent UTC timestamps independent of host system locale or timezone offsets.
- **Multiple Output Types:** Choose between numeric Unix timestamps (`number`) and formatted date strings (`string`).
- **Standard Date Formats:** Supports ISO 8601 (`2026-08-24T15:25:00.000Z`), RFC 2822 (`Mon, 24 Aug 2026 15:25:00 GMT`), database formats (`YYYY-MM-DD`, `YYYY-MM-DD HH:mm:ss`), and localized formats (`DD/MM/YYYY`, `MM/DD/YYYY`).
- **Dynamic Parameter UI:** The `format` selector dynamically displays when `outputType` is set to `string`, keeping configuration straightforward.
- **Zero Latency:** Executes locally in memory with zero external API dependencies.

### Processing Flow

```text
Workflow Trigger / Incoming Data
  ↓
Read Output Type ('string' or 'number')
  ↓
If 'number': Return current Date.getTime() (Unix milliseconds)
  ↓
If 'string': Apply selected format (ISO 8601, RFC 2822, YYYY-MM-DD, etc.)
  ↓
Emit result to 'success' output
```

### Use Cases

- **File & Report Naming:** Generate dynamic filenames with timestamps (e.g. `report_2026-08-24.csv` or `backup_20260824_153000.json`).
- **Audit Logging & Records:** Stamp database rows or webhook payloads with exact creation/execution timestamps (`createdAt: 1787578072031`).
- **Time Range Queries:** Fetch the current time to construct start/end query intervals for database searches or API requests.
- **Cache Invalidation & TTL:** Compute expiration times and refresh tokens by capturing the current timestamp.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `outputType` | `enum` | No | `string` | The format category to output: `string` (formatted date string) or `number` (Unix timestamp in milliseconds). |
| `format` | `enum` | No | `ISO 8601` | The date string format used when `outputType` is `string`. Supported formats: `ISO 8601`, `RFC 2822`, `YYYY-MM-DD`, `YYYY-MM-DD HH:mm:ss`, `DD/MM/YYYY`, `MM/DD/YYYY`. |

### Available Formats Reference

| Format Option | Output Example | Typical Use Case |
|---------------|----------------|------------------|
| `ISO 8601` | `2026-08-24T15:25:00.000Z` | Standard API payloads, JSON records, and database timestamps |
| `RFC 2822` | `Mon, 24 Aug 2026 15:25:00 GMT` | HTTP headers, email headers, and RSS feeds |
| `YYYY-MM-DD` | `2026-08-24` | Database date columns, date partitions, file naming |
| `YYYY-MM-DD HH:mm:ss` | `2026-08-24 15:25:00` | SQL datetime fields, application logs |
| `DD/MM/YYYY` | `24/08/2026` | European and international date displays |
| `MM/DD/YYYY` | `08/24/2026` | US standard date displays |

### Default Configuration

```json
{
  "outputType": "string",
  "format": "ISO 8601"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or previous node payload passed to start the step. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` or `number` | The current UTC date/time in the selected format or timestamp value. |
| `error` | `object` | Returned if an unhandled internal error occurs. |

### Output Examples

#### String Output (ISO 8601)

```json
"2026-08-24T15:25:00.000Z"
```

#### String Output (YYYY-MM-DD)

```json
"2026-08-24"
```

#### Numeric Output (Unix Milliseconds)

```json
1787578072031
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: ISO 8601 Timestamp for API Requests

Generate a standard ISO 8601 string for REST API payloads or event logging:

```json
{
  "outputType": "string",
  "format": "ISO 8601"
}
```

**Output:**
```text
"2026-08-24T15:25:00.000Z"
```

### Example 2: Date for File Naming (YYYY-MM-DD)

Generate a clean date format suitable for file paths or export naming:

```json
{
  "outputType": "string",
  "format": "YYYY-MM-DD"
}
```

**Output:**
```text
"2026-08-24"
```

### Example 3: Unix Millisecond Timestamp

Get the raw epoch time in milliseconds for numerical math or database indexing:

```json
{
  "outputType": "number"
}
```

**Output:**
```text
1787578072031
```

### Example 4: Datetime for SQL Logging (YYYY-MM-DD HH:mm:ss)

```json
{
  "outputType": "string",
  "format": "YYYY-MM-DD HH:mm:ss"
}
```

**Output:**
```text
"2026-08-24 15:25:00"
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate current UTC timestamps in different formats
```

### Common Workflow Patterns

- **Record Creation Timestamp:** Manual Trigger → Now (`outputType: "number"`) → Database / Airtable (Insert new item with `createdAt`).
- **Dynamic File Export:** Schedule Trigger → Now (`format: "YYYY-MM-DD"`) → Create File (Name: `export_{{ $node.Now }}.csv`) → AWS S3 / Google Drive.
- **API Request Headers:** Trigger → Now (`format: "RFC 2822"`) → HTTP Request (Attach header `Date: {{ $node.Now }}`).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Format selector is hidden in the UI

**Cause:** The `outputType` parameter is set to `number`.

**Solution:** Set `outputType` to `string` in the node parameters to choose a specific date string format.

### Local timezone offset differs from output

**Cause:** The **Now** node intentionally outputs UTC time (Coordinated Universal Time) to prevent server location discrepancies.

**Solution:** If you need a specific local timezone (e.g. `America/New_York` or `Africa/Casablanca`), connect the output of the **Now** node to a [Convert Timezone](../convert-timezone/en.md) or [Format Date](../format-date/en.md) node.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Format Date](../format-date/en.md) - Parse and format custom date strings
- [Date Add](../date-add/en.md) - Add or subtract days, hours, or minutes from a date
- [Date Compare](../date-compare/en.md) - Compare two dates to determine chronological order
- [Convert Timezone](../convert-timezone/en.md) - Convert UTC dates into specific local timezones

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for Now node |

<!-- /SECTION: changelog -->

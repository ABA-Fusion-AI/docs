---
node_id: "to-csv"
title: "To CSV"
description: "Convert array of objects to CSV string."
category: "data-transformation-etl"
subcategory: "parsing-serialization"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - to-csv
  - csv
  - export
  - serialization
  - tabular-data
  - data-transformation
  - parsing-serialization
related_nodes:
  - parse-csv
  - stringify-json
  - create-file
  - excel-add-rows
---

<!-- SECTION: header -->
# To CSV

> **Category:** Data Transformation (ETL) | **Subcategory:** Parsing & Serialization | **Type:** Action Node

Convert an array of structured JSON objects into a formatted CSV string with custom delimiters and header control.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **To CSV** node serializes arrays of JavaScript/JSON objects into RFC 4180-compliant Comma-Separated Values (CSV) text. It automatically discovers all unique column keys across the provided records, handles missing or uneven object properties, escapes embedded double quotes, and generates formatted tabular data ready for file downloads, spreadsheet exports, or email attachments.

### Key Features

- **Automatic Column Detection:** Scans all objects in the input array to dynamically build a unified header row, ensuring no keys are omitted even if some records contain optional or missing fields.
- **Custom Delimiters:** Use standard commas (`,`), semicolons (`;`), pipes (`|`), tabs (`\t`), or any custom character as the column separator.
- **Header Row Toggle:** Enable or disable the first header row using the `includeHeaders` option.
- **RFC 4180 Escaping:** Automatically quotes all cell values and escapes internal quotes (`"` $\rightarrow$ `""`), preventing corrupt layouts when text contains commas, spaces, or quotes.
- **Dual Input Modes:** Accepts a JSON string array directly in the `Data` parameter or consumes dynamic object arrays passed from upstream database, API, or transform nodes.
- **Zero Latency:** Executes synchronously in-memory with instant serialization.

### Processing Flow

```text
Incoming Array of Objects (Parameter or Upstream Node)
  ↓
Validate that Input is an Array
  ↓
Collect all unique keys across all objects
  ↓
Format Header Row (if includeHeaders = true)
  ↓
Format Data Rows (map each object's values & escape quotes)
  ↓
Join lines with newlines (\n) & return CSV text string
```

### Use Cases

- **Report Generation & Exports:** Convert database query results (e.g. Postgres, MySQL, Supabase) into CSV files for business reports.
- **Email Attachments:** Create CSV data exports on the fly and attach them to outgoing automated emails via [Email Send](../email-send/en.md) or [Gmail](../gmail/en.md).
- **Cloud Storage Backup:** Convert API responses into CSV and store them in Amazon S3, Google Drive, or Azure Blob Storage.
- **Spreadsheet Integration:** Format internal workflow records for direct import into Microsoft Excel or Google Sheets.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | JSON string representing an array of objects (e.g., `[{"id": 1, "name": "Abdel"}]`). If omitted, the node converts the array passed from the upstream node. |
| `delimiter` | `string` | No | `,` | The delimiter character used to separate columns in each row (e.g., `,`, `;`, `|`, or `\t`). |
| `includeHeaders` | `boolean` | No | `true` | When `true`, includes a header row with column names as the first line of the CSV. |

### Common Delimiters Reference

| Delimiter | Name | Typical Usage |
|-----------|------|---------------|
| `,` | Comma | Standard CSV format for English/US locale applications |
| `;` | Semicolon | European CSV standard (avoids conflicts with decimal commas) |
| `\|` | Pipe | Pipe-delimited flat files used in data warehouses and legacy systems |
| `\t` | Tab | Tab-Separated Values (TSV) format for easy spreadsheet pasting |

### Default Configuration

```json
{
  "data": "[\n  {\"id\": 1, \"name\": \"Abdel\", \"role\": \"Developer\"},\n  {\"id\": 2, \"name\": \"Sara\", \"role\": \"Designer\"}\n]",
  "delimiter": ",",
  "includeHeaders": true
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `array<object>` or `unknown` | Array of objects to serialize into CSV. Used automatically when the `data` parameter is not set. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` | The formatted CSV text string. |
| `error` | `object` | Returned if input data is invalid, not an array, or contains unparseable JSON text. |

### Output Example

```csv
"id","name","role"
"1","Abdel","Developer"
"2","Sara","Designer"
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Standard CSV with Headers

Convert an array of user objects with default comma delimiter:

```json
{
  "data": "[\n  {\"id\": 1, \"name\": \"Abdel\", \"role\": \"Developer\"},\n  {\"id\": 2, \"name\": \"Sara\", \"role\": \"Designer\"}\n]",
  "delimiter": ",",
  "includeHeaders": true
}
```

**Output:**
```csv
"id","name","role"
"1","Abdel","Developer"
"2","Sara","Designer"
```

### Example 2: CSV without Headers

Generate raw data rows without a top header line:

```json
{
  "data": "[\n  {\"id\": 1, \"product\": \"Laptop\", \"price\": 1200},\n  {\"id\": 2, \"product\": \"Mouse\", \"price\": 25}\n]",
  "includeHeaders": false
}
```

**Output:**
```csv
"1","Laptop","1200"
"2","Mouse","25"
```

### Example 3: Semicolon Delimiter (European Format)

Use semicolons (`;`) for European decimal and localized systems:

```json
{
  "data": "[\n  {\"ville\": \"Casablanca\", \"code\": 20000},\n  {\"ville\": \"Rabat\", \"code\": 10000}\n]",
  "delimiter": ";"
}
```

**Output:**
```csv
"ville";"code"
"Casablanca";"20000"
"Rabat";"10000"
```

### Example 4: Pipe Delimiter (`|`)

Format text containing commas by separating fields with pipes:

```json
{
  "data": "[\n  {\"title\": \"Book, Part 1\", \"author\": \"John Doe\"},\n  {\"title\": \"Book, Part 2\", \"author\": \"Jane Doe\"}\n]",
  "delimiter": "|"
}
```

**Output:**
```csv
"title"|"author"
"Book, Part 1"|"John Doe"
"Book, Part 2"|"Jane Doe"
```

### Example 5: Handling Uneven Keys (Missing Properties)

When objects have different keys, the node unites all columns and leaves missing values blank:

```json
{
  "data": "[\n  {\"id\": 1, \"name\": \"Karim\", \"phone\": \"0600000000\"},\n  {\"id\": 2, \"name\": \"Yassine\"}\n]"
}
```

**Output:**
```csv
"id","name","phone"
"1","Karim","0600000000"
"2","Yassine",""
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert JSON arrays of objects to CSV strings with custom delimiters
```

### Common Workflow Patterns

- **Database Export to S3:** Postgres Query (Fetch Users) → To CSV (`includeHeaders: true`) → Create File (`users.csv`) → AWS S3 Upload.
- **Automated CSV Report Email:** Scheduled Trigger (Monday 9 AM) → HTTP Request (Fetch Invoices) → To CSV → Email Send (Attach generated CSV).
- **Data Pipeline Serialization:** Webhook (Array of Orders) → To CSV (`delimiter: ";"`) → FTP/SFTP Write File.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "Data must be an array for CSV conversion"

**Cause:** The incoming data or `data` parameter is a single object, string, or number instead of an array.

**Solution:** Wrap single objects in array brackets `[{ ... }]` or use upstream array-shaping nodes (e.g. `Filter Array` or `Function`) to return an array of objects.

### Error: "Config data is not valid array objects"

**Cause:** The string provided in the `Data` parameter contains syntax errors and cannot be parsed as valid JSON.

**Solution:** Ensure the text in `Data` is valid JSON syntax (e.g. using double quotes around property names and strings).

### Error: "Data is required for CSV conversion"

**Cause:** Neither the `data` parameter was specified nor did the upstream node output any data (`null` or `undefined`).

**Solution:** Verify upstream connections or configure test JSON data in the parameter field.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Parse CSV](../parse-csv/en.md) - Parse a CSV string back into an array of JSON objects
- [Stringify JSON](../stringify-json/en.md) - Convert JavaScript objects into JSON strings
- [Create File](../create-file/en.md) - Save generated CSV text as a file in the workspace
- [Excel Add Rows](../excel-add-rows/en.md) - Append tabular records directly into Microsoft Excel worksheets

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for To CSV node |

<!-- /SECTION: changelog -->

---
  node_id: "generate-base64-file"
  title: "Generate Base64 File"
  description: "Turn structured rows or raw content into a file (XLSX, CSV, HTML, JSON or TXT) and return it as a base64-encoded string."
  category: "storage-files"
  subcategory: "files-documents"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-07-21"
  author: "Fusion Team"
  tags:
    - file
    - base64
    - xlsx
    - csv
    - export
    - spreadsheet
    - no-code
  related_nodes:
    - convert-to-file
    - encode-base64
    - decode-base64
    - http-request
---

<!-- SECTION: overview -->
  # Generate Base64 File

  > **Category:** Storage & Files&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Builds a file from your workflow data and emits it as a base64 string. Five formats are supported — **XLSX**, **CSV**, **HTML**, **JSON** and **TXT** — from either structured row objects or raw content.

  Nothing is written to disk. The file exists only as base64 in the node's output, ready to attach to an email, upload to storage, or return from a webhook.

### Use Cases

- Export query results as a spreadsheet and attach it to an email
- Build a CSV from an API response and upload it to object storage
- Return a generated report directly from a webhook response
- Produce a styled XLSX invoice with per-cell fonts, fills and number formats
- Snapshot workflow data as JSON for archival

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Choosing a format

`format` is the only always-required field. Every other field appears or hides based on it, so you only ever see what applies.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `format` | `enum` | ✅ Yes | — | Output format: `xlsx`, `csv`, `html`, `json`, or `txt`. |

### Where the data comes from

Two input shapes exist. **`rows`** is a JSON array of flat objects — column headers are derived from the keys (the union across all rows, in first-seen order). **`content`** is a raw string written verbatim.

| Format | Data source | Field used |
|--------|-------------|------------|
| `csv` | always rows | `rows` |
| `xlsx` | `xlsxMode: "rows"` | `rows` |
| `xlsx` | `xlsxMode: "workbook"` | `workbook` |
| `html` / `json` | `dataSource: "rows"` | `rows` |
| `html` / `json` | `dataSource: "content"` | `content` |
| `txt` | always raw | `content` |

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `rows` | `string` | Conditional | — | JSON array of flat row objects, e.g. `[{"Name":"Mouad","Age":"20"}]`. Cell values must be a string, number, boolean, or null. |
| `content` | `string` | Conditional | — | Raw file content, written exactly as given. |
| `dataSource` | `enum` | ❌ No | `rows` | HTML/JSON only — generate from `rows` or write `content` verbatim. |

### XLSX

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `xlsxMode` | `enum` | ❌ No | `rows` | `rows` for a plain sheet from row objects; `workbook` for full control over sheets, cells and styling. |
| `workbook` | `string` | Conditional | — | JSON workbook definition — see below. Used when `xlsxMode` is `workbook`. |
| `xlsxCompression` | `boolean` | ❌ No | `true` | Compress the underlying zip archive. Off is faster to write and slightly more compatible with tools expecting uncompressed entries, but produces a noticeably larger file. |

The `workbook` shape, when you need styling:

```json
{
  "sheets": [
    {
      "name": "Invoice",
      "columns": [{ "width": 30 }, { "width": 12 }],
      "rows": [
        {
          "cells": [
            { "value": "Item", "font": { "bold": true } },
            { "value": "Total", "font": { "bold": true } }
          ]
        },
        {
          "cells": [
            { "value": "Consulting" },
            { "value": 1250, "numberFormat": "#,##0.00" }
          ]
        }
      ]
    }
  ]
}
```

Every cell is styled explicitly — there is no conditional or rule-based styling. `columns` is optional and index-aligned with each row's cells, so the first entry is column A. Supported per-cell keys are `value`, `font`, `fill`, `alignment`, `border` and `numberFormat`.

### CSV

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `separator` | `enum` | ❌ No | `,` | Field separator: `,`, `;` or `\|`. Use `;` for locales where Excel expects it. |
| `includeHeader` | `boolean` | ❌ No | `true` | Emit a header row derived from the row keys. |

Fields containing the separator, a double quote, or a line break are quoted automatically, and embedded quotes are doubled.

### HTML & JSON

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `includeHeader` | `boolean` | ❌ No | `true` | HTML from rows — include the `<thead>` row. |
| `jsonPretty` | `boolean` | ❌ No | `true` | JSON from rows — indent the output rather than emitting it compact. |

HTML generated from rows produces a plain `<table>`, with all values HTML-escaped.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Triggers generation. Reference upstream data in `rows`, `content` or `workbook` with expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The generated file |
| `error` | `Error` | Emitted when input is invalid or generation fails |

### Output Schema (`success`)

```json
{
  "base64": "UEsDBBQAAAAIAA..."
}
```

A single `base64` string. There is no filename or MIME type — set those on whichever node consumes the file.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: CSV from rows

**Configuration:**
```json
{
  "format": "csv",
  "rows": "{{ $node['query'].output.data }}",
  "separator": ",",
  "includeHeader": true
}
```

**Input rows:**
```json
[
  { "Name": "Alice", "Age": 30 },
  { "Name": "Bob", "Age": 25 }
]
```

**Decoded output:**
```
Name,Age
Alice,30
Bob,25
```

---

### Example 2: Simple spreadsheet

Same row data, no styling.

```json
{
  "format": "xlsx",
  "xlsxMode": "rows",
  "rows": "{{ $node['query'].output.data }}",
  "includeHeader": true
}
```

The header row is bolded automatically.

---

### Example 3: Styled workbook

```json
{
  "format": "xlsx",
  "xlsxMode": "workbook",
  "workbook": "{\"sheets\":[{\"name\":\"Report\",\"columns\":[{\"width\":24}],\"rows\":[{\"cells\":[{\"value\":\"Revenue\",\"font\":{\"bold\":true,\"size\":14},\"fill\":{\"type\":\"pattern\",\"pattern\":\"solid\",\"fgColor\":\"#DDEBF7\"}}]}]}]}"
}
```

---

### Example 4: Raw HTML passthrough

```json
{
  "format": "html",
  "dataSource": "content",
  "content": "<h1>Weekly Report</h1><p>All systems nominal.</p>"
}
```

With `dataSource: "content"` the string is written byte-for-byte — no escaping and no table wrapper.

---

### Example 5: Compact JSON

```json
{
  "format": "json",
  "dataSource": "rows",
  "rows": "{{ $node['fetch'].output.data }}",
  "jsonPretty": false
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate a CSV and inspect it
```

### Sample Workflow: Email a daily report

```json
{
  "nodes": [
    { "id": "schedule", "type": "interval" },
    {
      "id": "query",
      "type": "postgres-action",
      "config": { "operation": "Select", "query": "SELECT name, total FROM orders WHERE day = CURRENT_DATE" }
    },
    {
      "id": "build-file",
      "type": "generate-base64-file",
      "config": {
        "format": "xlsx",
        "xlsxMode": "rows",
        "rows": "{{ $node['query'].output.data }}",
        "includeHeader": true
      }
    },
    {
      "id": "send",
      "type": "nodemailer-action",
      "config": {
        "attachments": [
          {
            "filename": "daily-orders.xlsx",
            "content": "{{ $node['build-file'].output.base64 }}",
            "encoding": "base64"
          }
        ]
      }
    }
  ]
}
```

**How it flows:**
1. The interval trigger fires once a day.
2. Postgres returns an array of row objects.
3. **Generate Base64 File** turns them into an XLSX and emits `{ base64 }`.
4. The mail node attaches it — the filename lives here, not on the file node.

### Common Patterns

- **Export → upload:** feed `base64` straight into a storage node
- **Webhook download:** return the base64 from Respond to Webhook with a `Content-Disposition` header
- **Multi-format:** fan one dataset into parallel nodes for CSV and XLSX

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `` `rows` is not valid JSON ``

**Cause:** The expression produced an object or a JS value rather than a JSON string, or produced nothing.

**Solution:** `rows` must resolve to a JSON **string**. Wrap the upstream value in a stringify step if the node hands you a live object.

#### `` `rows[0].total` must be a string, number, boolean, or null ``

**Cause:** A cell held a nested object, an array, or a date instance.

**Solution:** Flatten before this node — one level of key/value per row. Format dates to strings upstream.

#### `` `rows` is required for this format ``

**Cause:** CSV always reads `rows`; HTML/JSON read it only when `dataSource` is `rows`, and XLSX only when `xlsxMode` is `rows`.

**Solution:** Check the mode toggle matches the field you filled in.

#### Spreadsheet opens with everything in one column

**Cause:** Excel in some locales expects `;` rather than `,`.

**Solution:** Set `separator` to `;`, or use `xlsx` instead of `csv`.

#### The workflow slows down or memory climbs on large exports

**Cause:** The whole file is built in memory and base64 adds roughly a third to its size. Turning `xlsxCompression` off makes this worse.

**Solution:** Batch large exports into several files, and leave compression on.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `` `content` is required for this format `` | TXT selected, or HTML/JSON with `dataSource: "content"`, but the field is empty | Fill `content`, or switch `dataSource` to `rows` |
| `` `rows` must be a JSON array of row objects `` | The JSON parsed to an object or scalar | Wrap it in an array |
| `` `rows[i]` must be a plain object `` | An array element was a scalar or nested array | Each row is one flat object |
| `` `workbook.sheets[i].name` must be a string `` | Sheet missing a name | Name every sheet |
| `` `workbook.sheets[i].columns[j].width` must be a number `` | Width given as a string | Use a number, e.g. `24` not `"24"` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Encode Base64](./encode-base64.md) – Encode an existing string or buffer
- [Decode Base64](./decode-base64.md) – Turn a base64 string back into content
- [HTTP Request](./http-request.md) – Upload the generated file to an external API

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-07-21 | Initial release — XLSX (rows + styled workbook), CSV, HTML, JSON and TXT |

<!-- /SECTION: changelog -->

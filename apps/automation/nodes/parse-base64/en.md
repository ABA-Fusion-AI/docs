---
node_id: "parse-base64"
title: "Parse Base64"
description: "Decode Base64 and parse to TXT, JSON, CSV, or XLSX."
category: "utilities"
subcategory: "Data Transformation"
version: "1.1.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - base64
  - decode
  - parser
  - json
  - csv
  - transform
related_nodes:
  - parse-json
  - parse-csv
  - function
  - encode-base64
---

<!-- SECTION: overview -->
# Parse Base64

> **Category:** Utilities &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Decode a Base64 string and parse the result as plain text, JSON, CSV, or XLSX. The node automatically strips `data:*;base64,` prefixes and handles both standard and URL-safe Base64 variants.

### Use Cases

- **File Decoding:** Decode Base64-encoded files received from APIs or webhooks and extract their content.
- **API Payloads:** Decode Base64 fields embedded in API responses (e.g., email attachments, encoded config values).
- **CSV Ingestion:** Decode and parse Base64-encoded CSV exports into structured arrays for further processing.
- **Pipeline Transformation:** Use as a middle step between a Function node that returns a Base64 string and downstream nodes that need parsed data.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | The Base64 string to decode. If omitted, the node reads from the incoming input (see auto-detection below). |
| `format` | `enum` | No | `txt` | Output format: `txt`, `json`, `csv`, or `xlsx`. |
| `delimiter` | `string` | No | `,` | Column separator for CSV parsing. Only used when `format` is `csv`. |
| `hasHeader` | `boolean` | No | `true` | Whether the first CSV row is a header. Only used when `format` is `csv`. |
| `encoding` | `enum` | No | `utf8` | Character encoding: `utf8`, `ascii`, or `latin1`. |

### Format Behavior

| Format | Output type | Notes |
|--------|-------------|-------|
| `txt` | `string` | Returns the decoded text as-is. |
| `json` | `object` or `array` | Parses the decoded string as JSON. Throws if invalid. |
| `csv` + `hasHeader: true` | `Array<Record<string, string>>` | First row becomes object keys. |
| `csv` + `hasHeader: false` | `string[][]` | Returns raw rows as arrays, including the header row. |
| `xlsx` | `object` | Parses the decoded binary as an Excel workbook. |

### Encoding Guide

| Encoding | Use when |
|----------|----------|
| `utf8` | **Recommended.** Works with JSON, CSV, TXT, Arabic, French, English, emojis. |
| `ascii` | Plain English/ASCII-only content. Does not support accented or non-Latin characters. |
| `latin1` | Legacy files with Western European characters (`é`, `à`, `ç`). No Arabic support. |

### Auto-Detection (when `data` is empty)

If the `data` parameter is not set, the node automatically checks the incoming input in this order:

1. Incoming value as a plain string
2. `input.base64`
3. `input.data`

This lets you pipe the output of a Function node directly into Parse Base64 without setting `data` manually.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | The Base64 string directly, or an object with a `base64` / `data` field. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string`, `object`, or `array` | The decoded and parsed result. Shape depends on `format`. |
| `error` | `Error` | Emitted on invalid Base64, decoding failure, or JSON/CSV parse error. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Decode Base64 from Function Node
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Function Node:** Returns the Base64 string `"SGVsbG8gRnVzaW9uIQ=="` as the output.
3. **Parse Base64 Node:** Receives the string, decodes it, and returns `"Hello Fusion!"` on the `success` output.
4. **Log Node:** Displays the decoded result.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: testing -->
## Testing the Node

### Test 1 — TXT

| Field | Value |
|-------|-------|
| `data` | `SGVsbG8gRnVzaW9uIQ==` |
| `format` | `txt` |
| `encoding` | `utf8` |

**Expected output:**
```
Hello Fusion!
```

---

### Test 2 — JSON

| Field | Value |
|-------|-------|
| `data` | `eyJuYW1lIjoiQWJkZWxraGFsZWsiLCJhZ2UiOjI0LCJjb3VudHJ5IjoiTW9yb2NjbyJ9` |
| `format` | `json` |
| `encoding` | `utf8` |

**Expected output:**
```json
{
  "name": "Abdelkhalek",
  "age": 24,
  "country": "Morocco"
}
```

---

### Test 3 — CSV with Header

| Field | Value |
|-------|-------|
| `data` | `bmFtZSxhZ2UKQWxpLDI1CkZhdGltYSwzMA==` |
| `format` | `csv` |
| `delimiter` | `,` |
| `hasHeader` | `true` |
| `encoding` | `utf8` |

**Expected output:**
```json
[
  { "name": "Ali", "age": "25" },
  { "name": "Fatima", "age": "30" }
]
```

---

### Test 4 — CSV without Header

Same Base64 as Test 3, but with `hasHeader` set to `false`.

**Expected output:**
```json
[
  ["name", "age"],
  ["Ali", "25"],
  ["Fatima", "30"]
]
```

---

### Test 5 — Input from Function Node

Use this workflow setup:

```
Manual Trigger → Function → Parse Base64 → Log
```

Function code:
```js
return {
  base64: "SGVsbG8gRnVzaW9uIQ=="
};
```

In Parse Base64: leave `data` **empty**, set `format` to `txt`.

The node automatically reads `input.base64` and outputs:
```
Hello Fusion!
```

<!-- /SECTION: testing -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Invalid Base64 string`
- **Cause:** The `data` value contains characters that are not valid Base64.
- **Solution:** Validate the source string. The node accepts standard Base64, URL-safe Base64 (`-`, `_`), and optional `data:*;base64,` prefixes.

#### `JSON parsing failed`
- **Cause:** The decoded content is not valid JSON.
- **Solution:** Verify the decoded string is well-formed JSON before choosing `format: json`. Use `format: txt` first to inspect the raw content.

#### Empty output / nothing on `success`
- **Cause:** The `data` field is empty and the incoming input is not a string, nor does it have a `base64` or `data` field.
- **Solution:** Either set `data` explicitly or ensure the upstream node returns a string or an object with `base64` / `data`.

#### Wrong characters in output
- **Cause:** Incorrect `encoding` selected (e.g., `ascii` used for Arabic or French content).
- **Solution:** Switch to `utf8`, which covers the vast majority of modern text formats.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Parse JSON](./parse-json.md) – Parse a raw JSON string into an object
- [Parse CSV](./parse-csv.md) – Parse a raw CSV string into an array
- [Encode Base64](./encode-base64.md) – Encode data to Base64
- [Function](./function.md) – Generate or transform a Base64 string before passing it to this node

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-08-06 | Expanded documentation with testing guide, encoding reference, and auto-detection behavior |
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->

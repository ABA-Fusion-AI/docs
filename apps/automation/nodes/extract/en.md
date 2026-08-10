node_id: "extract"
title: "Extract"
description: "Extract substrings or regular-expression capture groups from input strings for downstream processing."
category: "Data Transformation (ETL)"
subcategory: "Data Shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - string
  - regex
  - extract
  - text
related_nodes:
  - function
  - regex-match
  - log
---

<!-- SECTION: header -->
# Extract

> **Category:** Data Transformation (ETL) | **Subcategory:** text processing | **Type:** Action Node

Extracts substrings or regular-expression capture groups from an input string. Useful for parsing identifiers, tokens, or structured text fields within automation flows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Extract** node supports two primary extraction modes:

- `substring`: extract by start index and length (or start and end)
- `regex`: extract using a regular expression and return a specific capture group or all groups

The node accepts input as a string and returns the extracted value(s) on the `success` output. Parsing or match failures are emitted on the `error` output.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | `enum` | ✅ Yes | `substring` | `substring` or `regex` extraction mode |
| `value` | `string` | Conditional | — | Input string to process (if omitted, node reads `input.value`) |
| `start` | `number` | Conditional | `0` | Start index for `substring` mode (0-based) |
| `length` | `number` | Conditional | — | Number of characters to extract for `substring` mode; if omitted `end` is used |
| `end` | `number` | Conditional | — | End index (exclusive) for `substring` mode |
| `pattern` | `string` | Conditional | — | Regular expression pattern (without delimiters) for `regex` mode |
| `flags` | `string` | No | `""` | Regex flags (e.g., `i`, `g`) |
| `group` | `number|string` | No | `0` | Capture group index to return; use `0` to return the full match; `"all"` to return all groups as an array |
| `default` | `string` | No | `""` | Default value when no match is found |

### Notes on Indexing

- `start` and `end` use JavaScript-style 0-based indexing. Negative indices are supported (interpreted from the end) where applicable.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Node reads `input.value` when the `value` parameter is not provided |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | Extracted string or array of strings (when `group: "all"`) |
| `error` | `Error` | Emitted when extraction fails or parameters are invalid |

Example success payload (substring):

```json
"ABC123"
```

Example success payload (regex with groups):

```json
{"groups": ["ABC", "123"]}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Substring Example

**Configuration:**

```text
mode: substring
value: "Order-12345-USD"
start: 6
length: 5
```

**Output:** `"12345"`

---

### Regex Example: Capture ID

**Configuration:**

```text
mode: regex
value: "Order-12345-USD"
pattern: "Order-(\d+)-([A-Z]{3})"
flags: ""
group: 1
```

**Output:** `"12345"`

---

### Regex Example: All Groups

**Configuration:**

```text
mode: regex
value: "Order-12345-USD"
pattern: "Order-(\d+)-([A-Z]{3})"
flags: ""
group: "all"
```

**Output:**

```json
{"groups": ["12345", "USD"]}
```

---

## Notes

- When using `regex` mode with the global flag `g`, only use `group: "all"` or be aware that global matches return multiple full-match results.
- For performance-sensitive flows, prefer simple substring extraction when possible.
- Use `default` to provide fallback values for non-matching inputs.

<!-- /SECTION: examples -->
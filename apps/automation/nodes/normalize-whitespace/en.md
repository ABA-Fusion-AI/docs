---
node_id: "normalize-whitespace"
title: "Normalize Whitespace"
description: "Normalize whitespace and remove control characters."
category: "data-transformation-etl"
subcategory: "data-shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:
  - whitespace
  - text
  - normalization
  - cleanup
  - transformation
related_nodes:
  - default-fill
  - extract
  - restructure
---

<!-- SECTION: header -->
# Normalize Whitespace

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Normalize whitespace in text, remove control characters, and optionally preserve line breaks.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Normalize Whitespace** node cleans textual data by removing control characters and normalizing whitespace.

It supports two processing modes:

- Collapse all whitespace into single spaces.
- Preserve line breaks while normalizing spaces and tabs.

### Key Features

- Removes control characters from input text.
- Collapses repeated whitespace.
- Optionally preserves line breaks.
- Normalizes Windows and legacy line endings when line breaks are preserved.
- Removes unnecessary spaces at the beginning and end of lines.
- Supports configured data or incoming workflow data.
- Converts non-string input values to text before processing.

### Processing Flow

```text
Input
  ↓
Resolve input data
  ↓
Convert to string
  ↓
Remove control characters
  ↓
Normalize whitespace
  ↓
Return normalized string
```

### Use Cases

- Cleaning user-provided text.
- Normalizing text before storage.
- Preparing text for downstream processing.
- Removing unwanted spaces and tabs.
- Cleaning multiline text while preserving its structure.
- Removing control characters from imported data.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | Text to normalize. If empty, the node can use incoming workflow data. |
| `preserveLineBreaks` | `boolean` | No | `false` | Preserve line breaks while normalizing spaces and tabs. |

### Data

Provide the text that should be normalized.

Example:

```text
Hello     world

This    is     Fusion
```

### Preserve Line Breaks

When `preserveLineBreaks` is `false`, all whitespace, including line breaks, is collapsed into single spaces.

Input:

```text
Hello     world

This    is     Fusion
```

Output:

```text
Hello world This is Fusion
```

When `preserveLineBreaks` is `true`, line breaks are preserved while spaces and tabs are normalized.

Output:

```text
Hello world

This is Fusion
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts text from the `data` parameter or from incoming workflow data.

Input resolution follows this order:

1. If incoming workflow data is an array, the incoming array is used.
2. Otherwise, if `data` contains a non-empty value, the configured `data` is used.
3. Otherwise, the incoming workflow data is used.

### Input Conversion

Before normalization, the input is converted to a string:

- Strings are used directly.
- Objects and arrays are converted using `JSON.stringify()`.
- Other values are converted using `String()`.

### Output

The node returns the normalized text directly as a string.

Example:

```text
Hello world This is Fusion
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Normalize All Whitespace

**Input**

```text
Hello     world

This    is     Fusion
```

**Configuration**

```text
preserveLineBreaks: false
```

**Output**

```text
Hello world This is Fusion
```

### Example 2: Preserve Line Breaks

**Input**

```text
Hello     world

This    is     Fusion
```

**Configuration**

```text
preserveLineBreaks: true
```

**Output**

```text
Hello world

This is Fusion
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Normalize Whitespace Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Data is required for whitespace normalization

**Cause:** No configured data or incoming workflow data was provided.

**Solution:** Provide a value in `data` or connect a previous node that returns input data.

### Line Breaks Are Removed

**Cause:** `preserveLineBreaks` is set to `false`.

**Solution:** Set `preserveLineBreaks` to `true` if line breaks need to be preserved.

### Object or Array Input Returns JSON Text

This is expected behavior. Objects and arrays are converted using `JSON.stringify()` before whitespace normalization.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Default Fill** — Fill missing values with defaults.
- **Extract** — Extract values from incoming data.
- **Restructure** — Restructure data before or after normalization.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-12 | Initial documentation for the Normalize Whitespace node. |

<!-- /SECTION: changelog -->
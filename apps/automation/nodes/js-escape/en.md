---
node_id: "js-escape"
title: "JS Escape"
description: "JS-string-escape for safe output in JavaScript contexts."
category: "data-transformation-etl"
subcategory: "encoding-hashing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - javascript
  - escape
  - string
  - encoding
  - text
related_nodes:
  - html-escape
  - url-encode
  - text-to-binary
---

<!-- SECTION: header -->
# JS Escape

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Escape special characters in text for safer use in JavaScript string contexts.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **JS Escape** node converts input data into a JavaScript-safe string by escaping special characters.

The node escapes:

- backslashes;
- single quotes;
- double quotes;
- line feeds;
- carriage returns;
- tab characters.

It can use configured text or incoming workflow data.

### Key Features

- Escapes JavaScript-sensitive characters.
- Escapes backslashes as `\\`.
- Escapes single quotes as `\'`.
- Escapes double quotes as `\"`.
- Converts newline characters to `\n`.
- Converts carriage returns to `\r`.
- Converts tab characters to `\t`.
- Supports incoming workflow data.
- Converts objects and arrays using `JSON.stringify()`.

### Processing Flow

```text
Input
  ↓
Resolve configured or incoming data
  ↓
Convert input to string
  ↓
Escape special characters
  ↓
Return escaped string
```

### Use Cases

- Preparing strings for JavaScript output.
- Escaping quotes in generated text.
- Escaping filesystem paths containing backslashes.
- Converting multiline text into escaped string content.
- Preparing text for downstream JavaScript processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | Text to escape. If empty, the node can use incoming workflow data. |

### Data

Provide the text that should be escaped.

Example:

```text
Hello "Fusion" it's a test
```

The node returns:

```text
Hello \"Fusion\" it\'s a test
```

### Input Priority

Input is resolved in this order:

1. If incoming workflow data is an array, the incoming array is used.
2. Otherwise, if configured `data` contains a non-empty value, configured `data` is used.
3. Otherwise, incoming workflow data is used.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts text from the `data` parameter or from incoming workflow data.

### Input Conversion

Before escaping, input values are converted to strings:

- Strings are used directly.
- Objects and arrays are converted using `JSON.stringify()`.
- Other values are converted using `String()`.

### Output

The node returns the escaped value directly as a string.

Example:

```text
C:\\Users\\Elite\\Documents
```

There is no wrapper object around the returned value.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Escape Quotes

**Input**

```text
Hello "Fusion" it's a test
```

**Output**

```text
Hello \"Fusion\" it\'s a test
```

### Example 2: Escape Backslashes

**Input**

```text
C:\Users\Elite\Documents
```

**Output**

```text
C:\\Users\\Elite\\Documents
```

### Example 3: Combined Characters

**Input**

```text
It's "Fusion" C:\test
```

**Output**

```text
It\'s \"Fusion\" C:\\test
```

### Example 4: Escape Tab Character

**Input**

The input contains a real tab character between `Fusion` and `AI`.

```text
Hello Fusion	AI
```

**Output**

```text
Hello Fusion\tAI
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: JS Escape Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Data is required for JS escaping

**Cause:** No configured `data` and no incoming workflow data were available.

**Solution:** Provide a value in `data` or connect a previous node that returns input data.

### Backslashes Appear Doubled

This is expected behavior.

A single backslash:

```text
\
```

is converted to:

```text
\\
```

for safe JavaScript string output.

### Quotes Are Prefixed with Backslashes

This is expected behavior.

The node escapes:

```text
'
"
```

as:

```text
\'
\"
```

### Object or Array Input Returns JSON Text

Objects and arrays are converted using `JSON.stringify()` before escaping.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **HTML Escape** — Escape HTML-sensitive characters.
- **URL Encode** — Encode text for URL contexts.
- **Text to Binary** — Convert textual data to binary representation.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-13 | Initial documentation for the JS Escape node. |

<!-- /SECTION: changelog -->
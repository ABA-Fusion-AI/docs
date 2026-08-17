---
node_id: "url-encode"
title: "URL Encode"
description: "URL-encode string (encode or encodeComponent mode)."
category: "data-transformation-etl"
subcategory: "encoding-hashing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - url
  - encode
  - encoding
  - uri
  - text
related_nodes:
  - html-escape
  - js-escape
  - text-to-binary
---

<!-- SECTION: header -->
# URL Encode

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Encode text for use in URLs using either URI encoding or URI component encoding.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **URL Encode** node encodes input data for use in URLs.

It supports two encoding modes:

- `encode` — uses `encodeURI()`.
- `encodeComponent` — uses `encodeURIComponent()`.

### Key Features

- Encodes spaces and special characters for URL usage.
- Supports complete URL encoding with `encodeURI()`.
- Supports URL component encoding with `encodeURIComponent()`.
- Preserves URL structural characters in `encode` mode.
- Encodes reserved URL characters in `encodeComponent` mode.
- Supports configured values or incoming workflow data.
- Converts non-string input values to text before encoding.
- Can extract a string from the `data` property of incoming objects.

### Processing Flow

```text
Input
  ↓
Resolve configured or incoming data
  ↓
Convert input to string
  ↓
Select encoding mode
  ↓
Encode value
  ↓
Return encoded string
```

### Use Cases

- Encoding complete URLs.
- Encoding query parameter values.
- Preparing dynamic URL components.
- Encoding text containing spaces or special characters.
- Preparing workflow data for use in URLs.
- Preparing values for downstream HTTP operations.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `value` | `string` | No | — | Value to encode. If empty, the node uses incoming workflow data. |
| `mode` | `string` | No | `encode` | Encoding mode: `encode` or `encodeComponent`. |

### Value

Provide the value that should be encoded.

Example:

```text
https://example.com/search?q=hello world&lang=fr
```

### Mode

The node supports two encoding modes.

#### encode

When `mode` is set to `encode`, the node uses `encodeURI()`.

URL structural characters such as `:`, `/`, `?`, `=` and `&` are preserved.

Input:

```text
https://example.com/search?q=hello world&lang=fr
```

Output:

```text
https://example.com/search?q=hello%20world&lang=fr
```

#### encodeComponent

When `mode` is set to `encodeComponent`, the node uses `encodeURIComponent()`.

Reserved URL characters are also encoded.

Input:

```text
https://example.com/search?q=hello world&lang=fr
```

Output:

```text
https%3A%2F%2Fexample.com%2Fsearch%3Fq%3Dhello%20world%26lang%3Dfr
```

### Input Priority

Input is resolved in this order:

1. If configured `value` is empty, `undefined`, or `null`, incoming workflow data is used.
2. Otherwise, configured `value` is used.

A configured string containing only whitespace is treated as empty.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts data from the `value` parameter or from incoming workflow data.

### Input Conversion

Before encoding, input values are converted to strings:

- Strings are used directly.
- If an object contains a string `data` property, that property is used.
- Other objects are converted using `JSON.stringify()`.
- Other values are converted using `String()`.

### Output

The node returns the encoded value directly as a string.

Example:

```text
https%3A%2F%2Fexample.com%2Fsearch%3Fq%3Dhello%20world%26lang%3Dfr
```

There is no wrapper object around the returned value.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Encode a Complete URL

**Input**

```text
https://example.com/search?q=hello world&lang=fr
```

**Configuration**

```text
mode: encode
```

**Output**

```text
https://example.com/search?q=hello%20world&lang=fr
```

### Example 2: Encode a URL Component

**Input**

```text
https://example.com/search?q=hello world&lang=fr
```

**Configuration**

```text
mode: encodeComponent
```

**Output**

```text
https%3A%2F%2Fexample.com%2Fsearch%3Fq%3Dhello%20world%26lang%3Dfr
```

### Example 3: Encode Incoming Data

If `value` is empty and the incoming workflow data is:

```text
Fusion AI & automation
```

with:

```text
mode: encodeComponent
```

the output is:

```text
Fusion%20AI%20%26%20automation
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: URL Encode Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Data is required for URL encoding

**Cause:** No configured value or incoming workflow data was provided.

**Solution:** Provide a value in `value` or connect a previous node that returns input data.

### URL Structural Characters Are Not Encoded

**Cause:** `mode` is set to `encode`.

This mode uses `encodeURI()` and preserves URL structural characters such as `:`, `/`, `?`, `=` and `&`.

**Solution:** Use `encodeComponent` when these characters also need to be encoded.

### Reserved URL Characters Are Encoded

This is expected behavior when `mode` is set to `encodeComponent`.

For example:

```text
https://example.com/?q=Fusion AI
```

is converted to:

```text
https%3A%2F%2Fexample.com%2F%3Fq%3DFusion%20AI
```

### Object Input Returns JSON Text

If an incoming object does not contain a string `data` property, the object is converted using `JSON.stringify()` before URL encoding.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **HTML Escape** — Escape HTML-sensitive characters.
- **JS Escape** — Escape special characters for JavaScript string contexts.
- **Text to Binary** — Convert textual data to binary representation.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the URL Encode node. |

<!-- /SECTION: changelog -->
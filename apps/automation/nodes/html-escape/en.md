---
node_id: "html-escape"
title: "HTML Escape"
description: "HTML-escape string for safe output."
category: "data-transformation-etl"
subcategory: "encoding-hashing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - html
  - escape
  - encoding
  - text
  - security
related_nodes:
  - js-escape
  - url-encode
  - text-to-binary
---

<!-- SECTION: header -->
# HTML Escape

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Escape HTML-sensitive characters in text for safer output in HTML contexts.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **HTML Escape** node converts input data into an HTML-safe string by replacing HTML-sensitive characters with their corresponding HTML entities.

The node escapes:

- Ampersands.
- Less-than signs.
- Greater-than signs.
- Double quotes.
- Single quotes.

### Key Features

- Escapes HTML-sensitive characters.
- Converts `&` to `&amp;`.
- Converts `<` to `&lt;`.
- Converts `>` to `&gt;`.
- Converts `"` to `&quot;`.
- Converts `'` to `&#39;`.
- Supports configured data or incoming workflow data.
- Converts non-string input values to text before processing.
- Returns the escaped text directly as a string.

### Processing Flow

```text
Input
  ↓
Resolve input data
  ↓
Convert to string
  ↓
Escape HTML-sensitive characters
  ↓
Return escaped string
```

### Use Cases

- Preparing text for HTML output.
- Escaping user-provided text.
- Preventing HTML characters from being interpreted as markup.
- Preparing text for downstream HTML processing.
- Escaping quotes and special characters before rendering.

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
<p class="test">Fusion & AI's platform</p>
```

The node returns:

```text
&lt;p class=&quot;test&quot;&gt;Fusion &amp; AI&#39;s platform&lt;/p&gt;
```

### Input Priority

Input is resolved in this order:

1. If incoming workflow data is an array, the incoming array is used.
2. Otherwise, if `data` contains a non-empty value, the configured `data` is used.
3. Otherwise, the incoming workflow data is used.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts text from the `data` parameter or from incoming workflow data.

### Input Conversion

Before escaping, the input is converted to a string:

- Strings are used directly.
- Objects and arrays are converted using `JSON.stringify()`.
- Other values are converted using `String()`.

### Output

The node returns the escaped text directly as a string.

Example:

```text
&lt;div&gt;Fusion &amp; AI&lt;/div&gt;
```

There is no wrapper object around the returned value.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Escape HTML Characters

**Input**

```text
&<>"'
```

**Output**

```text
&amp;&lt;&gt;&quot;&#39;
```

### Example 2: Escape HTML Markup

**Input**

```text
<p class="test">Fusion & AI's platform</p>
```

**Output**

```text
&lt;p class=&quot;test&quot;&gt;Fusion &amp; AI&#39;s platform&lt;/p&gt;
```

### Example 3: Escape Repeated Characters

**Input**

```text
<<<<Fusion & AI>>>>
```

**Output**

```text
&lt;&lt;&lt;&lt;Fusion &amp; AI&gt;&gt;&gt;&gt;
```

### Example 4: Already Escaped Content

**Input**

```text
&lt;div&gt;
```

**Output**

```text
&amp;lt;div&amp;gt;
```

Existing HTML entities are escaped again because the node processes every `&` character.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: HTML Escape Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Data is required for HTML escaping

**Cause:** No configured data or incoming workflow data was provided.

**Solution:** Provide a value in `data` or connect a previous node that returns input data.

### HTML Entities Appear as Characters in the Log

The Log interface may render HTML entities when displaying the output.

For example, the escaped value:

```text
&lt;div&gt;
```

may visually appear as:

```text
<div>
```

This does not necessarily mean that the HTML Escape node returned unescaped content.

### Existing HTML Entities Are Escaped Again

This is expected behavior.

For example:

```text
&lt;
```

is converted to:

```text
&amp;lt;
```

because the node escapes every ampersand.

### Object or Array Input Returns JSON Text

This is expected behavior. Objects and arrays are converted using `JSON.stringify()` before HTML escaping.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **JS Escape** — Escape special characters for JavaScript string contexts.
- **URL Encode** — Encode text for URL contexts.
- **Text to Binary** — Convert textual data to binary representation.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-13 | Initial documentation for the HTML Escape node. |

<!-- /SECTION: changelog -->
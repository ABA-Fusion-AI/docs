---
node_id: "to-boolean"
title: "To Boolean"
description: "Convert value to boolean with optional strict mode."
category: "data-transformation-etl"
subcategory: "data-shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - to-boolean
  - boolean
  - type-conversion
  - cast
  - strict-mode
  - data-shaping
  - data-transformation
related_nodes:
  - to-number
  - to-string
  - if-else
  - default-fill
---

<!-- SECTION: header -->
# To Boolean

> **Category:** Data Transformation (ETL) | **Subcategory:** Data Shaping | **Type:** Action Node

Convert input strings, numbers, and dynamic values into boolean `true` or `false`, with optional strict validation.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **To Boolean** node converts various data types (such as text strings, numbers, booleans, and null/undefined values) into a standard boolean (`true` or `false`). It provides two operational modes:

- **Lenient Mode (Default):** Understands common human-readable flags like `"yes"`, `"no"`, `"on"`, `"off"`, `"true"`, `"false"`, `1`, and `0`. Empty strings and `null`/`undefined` resolve to `false`.
- **Strict Mode (`strict: true`):** Enforces strict binary inputs (`true`, `false`, `1`, `0`, `"true"`, `"false"`, `"1"`, `"0"`). Any ambiguous string (such as `"on"`, `"active"`, or `"yes"`) throws an error.

It is widely used to prepare condition flags for branching nodes (like [If / Else](../if-else/en.md)), normalize form inputs, or sanitize boolean database fields.

### Key Features

- **Natural Language Parsing:** Automatically converts `"yes"`, `"no"`, `"on"`, `"off"`, `"true"`, and `"false"` (case-insensitive and whitespace-trimmed).
- **Strict Mode Enforcement:** Guarantees that only valid binary representations are accepted.
- **Null-Safe:** Safely returns `false` when input is `null`, `undefined`, or empty without throwing runtime exceptions.
- **Dynamic & Direct Input:** Accepts values passed from upstream workflow nodes or hardcoded in the `Data` parameter.
- **Zero Latency:** Executes synchronously in memory with instant evaluation.

### Processing Flow

```text
Incoming Value (Parameter or Upstream Node)
  ↓
Is value null or undefined? → Return false
  ↓
Is value already a boolean? → Return boolean
  ↓
Is Strict Mode enabled?
  ├─ Yes → If value is true/1/"true"/"1" → true
  │        If value is false/0/"false"/"0" → false
  │        Otherwise → Throw "Cannot convert to boolean in strict mode"
  └─ No (Lenient):
       ├─ String check (trimmed, lowercase):
       │    "true", "1", "yes", "on" → true
       │    "false", "0", "no", "off", "" → false
       │    Other non-empty string → true
       ├─ Number check: 0 / NaN → false, any non-zero → true
       └─ Objects / Arrays → true
```

### Use Cases

- **Workflow Branching:** Sanitize incoming webhook payload flags before connecting them to an [If / Else](../if-else/en.md) node.
- **Form Checkboxes & Toggles:** Normalize form checkbox values (`"on"`, `"yes"`, `"true"`) into clean boolean types for APIs and databases.
- **Feature Flag Parsing:** Convert environment variables or configuration strings into active/inactive state flags.
- **Data Pipeline Sanitization:** Ensure boolean database columns receive real `true`/`false` primitives rather than arbitrary strings.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | The value or string to convert to boolean. If omitted, the node uses incoming data from the upstream workflow node. |
| `strict` | `boolean` | No | `false` | When `true`, restricts valid inputs strictly to `true`, `false`, `1`, `0`, `"true"`, `"false"`, `"1"`, and `"0"`. Rejects other strings with an error. |

### Lenient Mode Conversion Matrix (`strict: false`)

| Input Value | Type | Output Boolean | Note |
|-------------|------|----------------|------|
| `"true"`, `"TRUE"`, `" True "` | `string` | `true` | Case-insensitive & trimmed |
| `"yes"`, `"YES"`, `"y"` | `string` | `true` | Natural language true |
| `"on"`, `"ON"` | `string` | `true` | Switch/toggle state |
| `"1"`, `1` | `string` / `number` | `true` | Binary flag |
| `42`, `-5` | `number` | `true` | Any non-zero number |
| `"false"`, `"FALSE"` | `string` | `false` | Explicit false string |
| `"no"`, `"NO"` | `string` | `false` | Natural language false |
| `"off"`, `"OFF"` | `string` | `false` | Switch/toggle state |
| `"0"`, `0` | `string` / `number` | `false` | Zero is false |
| `""` (empty string) | `string` | `false` | Empty text is false |
| `null` / `undefined` | `unknown` | `false` | Missing data defaults to false |
| `"any other string"` | `string` | `true` | Standard JavaScript truthy string |

### Strict Mode Conversion Matrix (`strict: true`)

| Input Value | Output Boolean | Note |
|-------------|----------------|------|
| `true`, `1`, `"true"`, `"1"` | `true` | Valid strict truthy |
| `false`, `0`, `"false"`, `"0"` | `false` | Valid strict falsy |
| `"yes"`, `"on"`, `"active"`, `"2"` | `Error` | Throws `Cannot convert "..." to boolean in strict mode` |

### Default Configuration

```json
{
  "data": "yes",
  "strict": false
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or previous node payload passed to convert if `data` parameter is empty. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `boolean` | Returns `true` or `false`. |
| `error` | `object` | Returned if `strict` mode is enabled and the input value is not a valid strict boolean. |

### Output Example

```json
true
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Convert "yes" / "no" Strings

Convert human-friendly toggle strings to boolean:

```json
{
  "data": "yes"
}
```

**Output:**
```json
true
```

```json
{
  "data": "no"
}
```

**Output:**
```json
false
```

### Example 2: Convert "true" / "false" Text

```json
{
  "data": "false"
}
```

**Output:**
```json
false
```

### Example 3: Convert Arbitrary Text to Truthy

In default lenient mode, non-empty text strings evaluate to `true`:

```json
{
  "data": "[[ active_user ]]"
}
```

**Output:**
```json
true
```

### Example 4: Strict Mode Validation (Valid)

Enforce strict validation for numeric binary flags:

```json
{
  "data": "1",
  "strict": true
}
```

**Output:**
```json
true
```

### Example 5: Strict Mode Validation (Invalid)

Passing `"on"` when `strict: true` throws an error:

```json
{
  "data": "on",
  "strict": true
}
```

**Output (Error):**
```text
Cannot convert "on" to boolean in strict mode
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert strings and flags to boolean values in lenient and strict modes
```

### Common Workflow Patterns

- **Conditional Branching:** Webhook (`is_admin: "yes"`) → To Boolean → If/Else (`condition: {{ $node["To Boolean"] }}`) → Admin Path / User Path.
- **Form Data Cleaning:** Typeform Webhook (`newsletter: "on"`) → To Boolean → CRM / HubSpot (Set `subscribed: true`).
- **Feature Flag Evaluation:** Get Variable (`ENABLE_FEATURE`) → To Boolean (`strict: true`) → Feature Logic.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "Cannot convert '...' to boolean in strict mode"

**Cause:** The node has `strict: true` enabled, but the incoming value is not one of `true`, `false`, `1`, `0`, `"true"`, `"false"`, `"1"`, or `"0"` (for instance `"yes"`, `"on"`, or `"enabled"`).

**Solution:** Set `strict: false` if you want to support natural language flags like `"yes"` or `"on"`, or sanitize the input before passing it to strict validation.

### Empty strings evaluating to false

**Cause:** In lenient mode, empty string `""` is considered falsy and evaluates to `false`.

**Solution:** This is the intended behavior. If empty strings should default to true, set a fallback value before conversion.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [To Number](../to-number/en.md) - Convert strings and booleans into numeric values
- [To String](../to-string/en.md) - Convert booleans and numbers into text strings
- [If / Else](../if-else/en.md) - Branch workflow execution based on boolean conditions
- [Default Fill](../default-fill/en.md) - Replace null or undefined values with fallbacks

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for To Boolean node |

<!-- /SECTION: changelog -->

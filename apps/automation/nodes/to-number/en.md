---
node_id: "to-number"
title: "To Number"
description: "Convert value to number with optional default on failure."
category: "data-transformation-etl"
subcategory: "data-shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - to-number
  - type-conversion
  - parse-number
  - cast
  - data-shaping
  - transformation
related_nodes:
  - to-string
  - to-boolean
  - default-fill
  - clamp
---

<!-- SECTION: header -->
# To Number

> **Category:** Data Transformation (ETL) | **Subcategory:** Data Shaping | **Type:** Action Node

Convert input strings, booleans, and JSON-encoded values into standard numeric values with optional default fallbacks.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **To Number** node converts incoming data of various types (such as text strings, JSON numbers, and booleans) into standard JavaScript numeric values. It is commonly used in data pipelines to clean, cast, and validate incoming API payloads, webhook query parameters, or form fields before passing them to mathematical, financial, or database nodes.

If the incoming value is invalid, empty, or unparseable, the node can return a user-defined `defaultValue` instead of failing the workflow execution.

### Key Features

- **Flexible Input Sources:** Converts values provided directly in the `Data` parameter or dynamic values passed from upstream workflow nodes.
- **Type Coercion:** Supports integers (`"42"`), floating-point decimals (`"123.45"`), negative numbers (`"-15"`), scientific notation (`"1e5"`), and booleans (`true` $\rightarrow$ `1`, `false` $\rightarrow$ `0`).
- **JSON String Parsing:** Automatically parses JSON-encoded strings before numeric conversion.
- **Graceful Fallback:** Supports an optional `defaultValue` parameter to prevent workflow failures when inputs are null, empty, or non-numeric.
- **Zero Latency:** Executes synchronously in memory without external API dependencies.

### Processing Flow

```text
Incoming Data (Parameter or Upstream Node)
  ↓
Check if Input is JSON string (Parse if applicable)
  ↓
Is Input already a number? → Return number
  ↓
Is Input a string? → Trim whitespace → Convert via Number(trimmed)
  ↓
Is Input a boolean? → true: 1, false: 0
  ↓
Is Conversion successful?
  ├─ Yes → Return numeric result
  └─ No / NaN / Missing Data:
       ├─ Is defaultValue defined? → Return defaultValue
       └─ Otherwise → Throw conversion error
```

### Use Cases

- **Form & Webhook Sanitization:** Cast string numbers received from HTML forms or HTTP query parameters (e.g., `"?page=2&limit=50"`) into real numbers.
- **Financial & Calculation Pipelines:** Prepare numeric inputs before feeding them into calculators (e.g. [Profit Margin Calculator](../profit-margin-calculator/en.md) or [Transfer Pricing Calculator](../transfer-pricing/en.md)).
- **Database Type Casting:** Ensure fields match strict numeric SQL schema types before database inserts.
- **Safe Fallbacks:** Ensure loop counters or limits never evaluate to `NaN` or `undefined` by providing a fallback default (e.g., `defaultValue: 0`).

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | The value or string to convert to a number. If omitted, the node converts the incoming data received from the upstream workflow node. |
| `defaultValue` | `number` | No | — | Optional fallback number returned if conversion fails or if the input is null, empty, or invalid. |

### Type Conversion Rules

| Input Value | Type | Output Number | Explanation |
|-------------|------|---------------|-------------|
| `"42"` | `string` | `42` | Parsed as integer |
| `"123.45"` | `string` | `123.45` | Parsed as float |
| `"-99.9"` | `string` | `-99.9` | Negative numbers supported |
| `"  100  "` | `string` | `100` | Leading/trailing whitespace is trimmed |
| `true` | `boolean` | `1` | Boolean true converted to 1 |
| `false` | `boolean` | `0` | Boolean false converted to 0 |
| `150` | `number` | `150` | Returned directly without modification |
| `"abc"` | `string` | `defaultValue` or `Error` | Non-numeric string falls back to default value or throws error |
| `""` (empty) | `string` | `defaultValue` or `Error` | Empty string falls back to default value or throws error |
| `null` / `undefined` | `unknown` | `defaultValue` or `Error` | Missing data falls back to default value or throws error |

### Default Configuration

```json
{
  "data": "42"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or previous node payload. Used as the input value if `data` is not explicitly set in the parameters. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `number` | The converted numeric value (e.g. `42`, `123.45`, `0`). |
| `error` | `object` | Returned if conversion fails and no `defaultValue` is configured. |

### Output Examples

#### Integer Output

```json
42
```

#### Float Output

```json
123.45
```

#### Fallback Output

```json
0
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Convert String Integer

Convert a text representation of an integer to a number:

```json
{
  "data": "42"
}
```

**Output:**
```json
42
```

### Example 2: Convert Decimal String

Convert a string with decimal places:

```json
{
  "data": "123.45"
}
```

**Output:**
```json
123.45
```

### Example 3: Convert with Safe Default Fallback

Provide a fallback number in case the source value contains unparseable text:

```json
{
  "data": "invalid-text",
  "defaultValue": 0
}
```

**Output:**
```json
0
```

### Example 4: Convert Boolean to Number

Cast boolean states into integer flags (e.g. for database columns):

```json
{
  "data": "true"
}
```

**Output:**
```json
1
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert strings and dynamic payloads to numeric values
```

### Common Workflow Patterns

- **Webhook Query Parsing:** Webhook Trigger (`req.query.limit`) → To Number (`defaultValue: 10`) → Database Query (Apply limit).
- **Price Calculation Pipeline:** HTTP Request / Scraper (`"price": "$49.99"`) → Replace (Remove `$`) → To Number → Transfer Pricing / Profit Calculator.
- **CSV Data Cleaning:** Parse CSV → For-Each → To Number → Database Insert.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error: "Cannot convert '...' to number"

**Cause:** The input contains non-numeric characters (such as currency symbols `$`, letters, or punctuation) that cannot be parsed as a number.

**Solution:** Configure a `defaultValue` parameter to gracefully fall back on invalid inputs, or use a text cleaning node (such as [Replace](../replace/en.md)) to strip non-numeric characters before conversion.

### Error: "Cannot convert empty string to number"

**Cause:** An empty string `""` was passed without a `defaultValue`.

**Solution:** Provide a `defaultValue` (e.g. `0`) in the node configuration if empty strings are expected.

### Error: "Data is required for number conversion"

**Cause:** Neither the parameter `data` nor the upstream node payload provided a value (`undefined` or `null`).

**Solution:** Supply a value in `data`, verify upstream node output connections, or configure `defaultValue`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [To String](../to-string/en.md) - Convert numbers and objects into text strings
- [To Boolean](../to-boolean/en.md) - Convert strings and numbers into boolean true/false
- [Default Fill](../default-fill/en.md) - Fill missing or null fields with default values
- [Clamp](../clamp/en.md) - Restrict a numeric value within a minimum and maximum range

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for To Number node |

<!-- /SECTION: changelog -->

---
node_id: "base-converter"
title: "Number Base Converter"
description: "Convert between Binary, Octal, Decimal, Hexadecimal, and custom bases (2-36)."
category: "Data Transformation (ETL)"
subcategory: "data shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - number
  - base
  - conversion
  - transform
related_nodes:
  - function
  - math
  - log
---

<!-- SECTION: header -->
# Number Base Converter

> **Category:** Data Transformation (ETL) | **Subcategory:** data shaping | **Type:** Action Node

Convert numeric values between common bases (binary, octal, decimal, hexadecimal) and arbitrary bases between 2 and 36. The node accepts input as strings or numbers and outputs the converted representation.

<!-- /SECTION: header -->

---

## Overview

The **Number Base Converter** node supports conversions such as `toBinary`, `toDecimal`, `toHex`, and `custom` where you specify input and output bases. It handles integer values and provides optional padding and case controls for alphabetic digits in bases > 10.

### Features

- Support for bases 2 through 36
- Operations: `toBinary`, `toOctal`, `toDecimal`, `toHex`, `custom`
- Accepts input as numeric or string values
- Optional padding and uppercase control for output in bases > 10

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `toBinary` | Conversion operation: `toBinary`, `toOctal`, `toDecimal`, `toHex`, `custom` |
| `value` | `string|number` | Conditional | — | Value to convert. If omitted, the node reads `input.value`. |
| `inputBase` | `number` | Conditional | `10` | Base of the input value (required for `custom`) |
| `outputBase` | `number` | Conditional | — | Target base for `custom` operation (required for `custom`) |
| `pad` | `number` | No | `0` | Minimum width for output; pads with leading zeros if needed |
| `uppercase` | `boolean` | No | `true` | When true, alphabetic digits (A-Z) are uppercase in outputs for bases > 10 |

### Examples

Convert decimal 255 to hexadecimal:

```json
{
  "operation": "toHex",
  "value": 255
}
```

Custom conversion from base 16 to base 2:

```json
{
  "operation": "custom",
  "value": "FF",
  "inputBase": 16,
  "outputBase": 2
}
```

---

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Node reads `input.value` when the `value` parameter is not provided |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | `{ "input": {"value": "...","base": N}, "output": {"value": "...","base": M} }` |
| `error` | `Error` | Emitted when value parsing fails or bases are out of range |

Example success payload:

```json
{
  "input": { "value": "FF", "base": 16 },
  "output": { "value": "11111111", "base": 2 }
}
```

---

## Examples

1. Decimal to Binary

Configuration: `operation=toBinary`, `value=13` -> Output: `1101`

2. Hex to Decimal

Configuration: `operation=toDecimal`, `value=FF` -> Output: `255`

3. Custom Base Conversion

Configuration: `operation=custom`, `value=1011`, `inputBase=2`, `outputBase=10` -> Output: `11`

---

## Notes

- The node currently supports integer conversions only. Fractional conversions are not supported.
- Input validation will emit descriptive errors when characters are invalid for the declared `inputBase`.
- For large numbers beyond JavaScript safe integer limits, pass input as strings and use appropriate big-number handling downstream.

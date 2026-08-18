---
node_id: "roman-numeral-converter"
title: "Roman Numeral Converter"
description: "Bidirectional conversion between Arabic and Roman numerals with validation"
category: "Utility / Conversion"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:

- roman-numerals
- conversion
- validation
- utility
- numbers
- arabic-numerals

related_nodes:
- function
- if

---

# Roman Numeral Converter

> **Category:** utility-nodes | **Type:** Action Node

Convert between **Arabic and Roman numerals**, validate Roman numeral strings, and batch-convert a numeric range — entirely locally, with no external API calls.

The **Roman Numeral Converter** node exposes four operations: `toRoman`, `toArabic`, `validate`, and `convertRange`.

### Supported Features

- Convert an Arabic number (1–3999) to a Roman numeral
- Convert a Roman numeral string to its Arabic value
- Validate whether a string is a well-formed Roman numeral (beyond just character set)
- Batch-convert an inclusive numeric range to Roman numerals
- Pattern-based validation rejecting common malformed numerals (e.g. `IIII`, `VV`, `IC`)
- Graceful, non-throwing error responses — all operations return `{ success: false, error }` instead of throwing

### Use Cases

- Display page numbers, chapter numbers, or clock faces in Roman numeral style
- Validate user-submitted Roman numeral input (e.g. a form field)
- Parse a Roman numeral back into a number for calculation
- Generate a lookup table of Roman numerals for a fixed range (e.g. 1–20 for outline formatting)
- Build educational or trivia content involving numeral systems

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"toRoman"` | Operation to perform: `toRoman`, `toArabic`, `validate`, or `convertRange`. |
| `number` | `number` | ✅ Yes (for `toRoman`) | `100` | Arabic number to convert, between 1 and 3999. |
| `roman` | `string` | ✅ Yes (for `toArabic`, `validate`) | `""` | Roman numeral string to convert or validate. |
| `start` | `number` | ❌ No (for `convertRange`) | `1` | Start of the range (inclusive), between 1 and 3999. |
| `end` | `number` | ❌ No (for `convertRange`) | `10` | End of the range (inclusive), between 1 and 3999. |

Note: the schema does not use conditional (`dependsOn`) visibility for this node — all fields are always present, but only the ones relevant to the selected `operation` are actually used.

---

## Operations

| Operation | Required Input | Description |
| --------- | ---------------- | ----------- |
| `toRoman` | `number` | Convert an Arabic number to Roman numerals. |
| `toArabic` | `roman` | Convert a Roman numeral string to its Arabic value, including a validity check. |
| `validate` | `roman` | Check whether a string is a well-formed Roman numeral. |
| `convertRange` | `start`, `end` | Convert every number in `[start, end]` to Roman numerals. |

---

## Conversion Logic

### `toRoman`

Greedy subtraction against a fixed value/numeral table (`M`, `CM`, `D`, `CD`, `C`, `XC`, `L`, `XL`, `X`, `IX`, `V`, `IV`, `I`), from largest to smallest. Valid only for numbers strictly between 0 and 4000 (i.e. 1–3999).

### `toArabic`

Standard left-to-right summation with subtractive-pair detection: if a numeral's value is less than the value of the numeral immediately after it, it is subtracted instead of added (e.g. `IV` = 4). The input is uppercased and trimmed before parsing, and must consist only of `M`, `D`, `C`, `L`, `X`, `V`, `I` characters. The result also includes a `valid` field from the same pattern-based check used by `validate`.

### `validate`

Two-step check:
1. The string must contain only valid Roman numeral characters (`M`, `D`, `C`, `L`, `X`, `V`, `I`).
2. The string must not match any of a fixed list of invalid patterns: repeated-numeral overruns (`IIII`, `VV`, `XXXX`, `LL`, `CCCC`, `DD`, `MMMM`) and invalid subtractive combinations (`IL`, `IC`, `ID`, `IM`, `XD`, `XM`, `VX`, `VL`, `VC`, `VD`, `VM`).

A string can pass the character-set check but still be marked invalid due to malformed structure (e.g. `"IIII"` or `"IL"`).

### `convertRange`

Loops from `start` to `end` (inclusive), calling `toRoman` for each value and **silently skipping** any value that fails conversion (in practice, only out-of-range values within a valid `start`/`end` bound would fail, which itself is prevented by the parameter's `min`/`max` constraints).

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

Every operation returns a `success` boolean. On failure, `success: false` is returned with an `error` string — **the node never throws**; errors are always returned as normal output data.

#### `toRoman`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Whether the conversion succeeded. |
| `arabic` | `number` | The input number. |
| `roman` | `string` | The resulting Roman numeral. |
| `length` | `number` | Character length of the resulting numeral. |

#### `toArabic`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Whether the conversion succeeded. |
| `roman` | `string` | The cleaned (uppercased, trimmed) input numeral. |
| `arabic` | `number` | The resulting Arabic number. |
| `valid` | `boolean` | Whether the input is a well-formed Roman numeral (structural validity, not just character set). |

#### `validate`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` if a `roman` value was provided. |
| `roman` | `string` | The input string, as provided (not cleaned). |
| `valid` | `boolean` | Whether the input is a well-formed Roman numeral. |

#### `convertRange`

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true`. |
| `start` | `number` | The configured range start. |
| `end` | `number` | The configured range end. |
| `results` | `array` | List of `{ arabic, roman, length }` objects for each value in the range. |

---

## Output Example

### `toRoman`

```json
{
  "success": true,
  "arabic": 1994,
  "roman": "MCMXCIV",
  "length": 7
}
```

### `toArabic`

```json
{
  "success": true,
  "roman": "MCMXCIV",
  "arabic": 1994,
  "valid": true
}
```

### `validate`

```json
{
  "success": true,
  "roman": "IIII",
  "valid": false
}
```

### `convertRange`

```json
{
  "success": true,
  "start": 1,
  "end": 5,
  "results": [
    { "arabic": 1, "roman": "I", "length": 1 },
    { "arabic": 2, "roman": "II", "length": 2 },
    { "arabic": 3, "roman": "III", "length": 3 },
    { "arabic": 4, "roman": "IV", "length": 2 },
    { "arabic": 5, "roman": "V", "length": 1 }
  ]
}
```

---

## Configuration Examples

### Convert Number to Roman

```json
{
  "operation": "toRoman",
  "number": 1994
}
```

### Convert Roman to Arabic

```json
{
  "operation": "toArabic",
  "roman": "MCMXCIV"
}
```

### Validate a Roman Numeral

```json
{
  "operation": "validate",
  "roman": "IIII"
}
```

### Convert a Range

```json
{
  "operation": "convertRange",
  "start": 1,
  "end": 20
}
```

---

## Workflow Integration

### Sample Workflow: Convert → Function

```json
{
  "nodes": [
    {
      "id": "roman-convert",
      "type": "roman-numeral-converter",
      "config": {
        "operation": "toRoman",
        "number": 2026
      }
    },
    {
      "id": "format-output",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Validate → If

```json
{
  "nodes": [
    {
      "id": "roman-validate",
      "type": "roman-numeral-converter",
      "config": {
        "operation": "validate",
        "roman": "MCMXCIV"
      }
    },
    {
      "id": "check-valid",
      "type": "if"
    }
  ]
}
```

### Sample Workflow: Generate a Range for Display

```json
{
  "nodes": [
    {
      "id": "roman-range",
      "type": "roman-numeral-converter",
      "config": {
        "operation": "convertRange",
        "start": 1,
        "end": 12
      }
    },
    {
      "id": "build-clock-face",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- Roman Numeral Converter (`validate`) → If → Error Notification — form input validation
- Roman Numeral Converter (`convertRange`) → Function → Document/slide generation — chapter or outline numbering
- Function (extract number) → Roman Numeral Converter (`toRoman`) → Database — store display-formatted values

---

## Error Handling

The node **does not throw errors** — all failures are caught and returned as normal output with `success: false`.

### Missing Roman Numeral (`toArabic`, `validate`)

```json
{ "success": false, "error": "Roman numeral is required" }
```

### Invalid Operation

```json
{ "success": false, "error": "Invalid operation" }
```

Should not normally occur given the `operation` enum, but is returned defensively if an unrecognized value slips through.

### Conversion Errors

Any error thrown internally by `toRoman` or `toArabic` (e.g. `"Number must be between 1 and 3999"`, `"Invalid Roman numeral"`) is caught and returned as:

```json
{ "success": false, "error": "<message>" }
```

---

## Troubleshooting

### `success: false, error: "Roman numeral is required"`

**Cause**

`roman` was left empty for `toArabic` or `validate`.

**Solution**

Provide a non-empty `roman` value.

---

### `success: false, error: "Invalid Roman numeral"`

**Cause**

The `roman` string, for `toArabic`, contains characters outside `M`, `D`, `C`, `L`, `X`, `V`, `I` (after uppercasing/trimming).

**Solution**

Check the input for typos or non-Roman characters (numbers, punctuation, other letters).

---

### `success: false, error: "Number must be between 1 and 3999"`

**Cause**

`number` (for `toRoman`), or a value implicitly generated inside `convertRange`, falls outside the 1–3999 range supported by standard Roman numerals.

**Solution**

Use a value within 1–3999 — Roman numerals as implemented here have no representation for 0 or numbers ≥ 4000.

---

### `toArabic` Succeeds but `valid` is `false`

**Cause**

The input contains only valid Roman characters but violates numeral construction rules (e.g. `"IIII"`, `"VV"`, `"IL"`) — the node still computes a best-effort Arabic value, but flags it as structurally invalid.

**Solution**

Treat `valid: false` results with caution; don't trust `arabic` values from malformed input for downstream logic without checking `valid` first.

---

### `convertRange` Returns Fewer Results Than Expected

**Cause**

Any value in `[start, end]` that fails `toRoman` internally is silently skipped rather than included as an error entry — in practice this should only happen if `start`/`end` somehow bypass the 1–3999 schema constraint.

**Solution**

Confirm `start` and `end` are both within 1–3999 and that `start <= end`.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally from the provided configuration values.

No API key or authentication credential is required.

---

## Notes

The node is designed to **never throw** — every operation, including internal conversion errors, is caught and returned as a normal `{ success: false, error }` output object, so downstream nodes should check `success` rather than relying on workflow-level error handling.

The node does not:

- Support numbers outside 1–3999 (no representation for 0, negatives, or numbers ≥ 4000)
- Support lowercase or mixed-case Roman numeral input beyond automatic uppercasing in `toArabic`
- Support alternative or non-standard numeral notations (e.g. vinculum/overline notation for large numbers)
- Return partial results for `convertRange` errors — failing values are simply omitted, not flagged

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-18 | Initial release |
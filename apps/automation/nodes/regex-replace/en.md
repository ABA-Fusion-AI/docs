---
node_id: "regex-replace"
title: "Regex Replace"
description: "Replace text using regex pattern."
category: "Utility / Text Processing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- regex
- regular-expressions
- text-replacement
- string-manipulation
- text-processing
- utility

related_nodes:
- regex-match
- function

---

# Regex Replace

> **Category:** utility-nodes | **Type:** Action Node

Replace text in a string using a **regular expression pattern** and a replacement string.

The **Regex Replace** node builds a JavaScript `RegExp` from a configured pattern and flags, applies it to input data (from config or the workflow input), and returns the resulting string after replacement.

### Supported Features

- Replace matches of a custom regex pattern with a replacement string
- Configurable regex flags (e.g. `g`, `i`, `m`)
- Support for replacement patterns with capture group references (e.g. `$1`, `$2`)
- Accepts input data from either node configuration or upstream workflow data
- Automatic array pass-through: if the upstream input is already an array, it is used directly as the target (stringified via `JSON.stringify` before replacement)
- Automatic stringification of non-string, non-array input
- Defaults to global replacement (`g` flag) when no flags are specified

### Use Cases

- Redact or mask sensitive data (e.g. replace digits in a phone number with `*`)
- Normalize text formatting (e.g. collapse multiple spaces, strip HTML tags)
- Rewrite structured tokens using capture groups (e.g. reformat dates)
- Clean up scraped or user-submitted text before storage or display
- Sanitize text before sending it to an LLM or external API

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `data` | `string` | ❌ No | — | String to perform replacement on. Used when set and non-empty; otherwise falls back to the node's input data. |
| `pattern` | `string` | ✅ Yes | — | Regular expression pattern (without delimiters), e.g. `\\s+` or `\\d{3}-\\d{4}`. |
| `replacement` | `string` | ✅ Yes | — | Replacement string. Supports `$1`, `$2`, etc. to reference capture groups from `pattern`, and `$&` for the whole match. |
| `flags` | `string` | ❌ No | `"g"` | Regex flags, e.g. `gi`, `m`. Defaults to `"g"` (global) if omitted. |

---

## Input Resolution

The string used for replacement is resolved in this order:

1. **If the workflow input `data` is an array** — it is used directly as `inputData`, regardless of the `data` config field.
2. **Else if the config field `data` is set and non-empty** (`!== undefined`, `!== null`, `!== ""`) — the config value is used.
3. **Else** — the workflow's input `data` is used as-is.

The resolved `inputData` is then converted to a string for replacement:
- A `string` is used as-is.
- An `object` (including an array from step 1) is converted via `JSON.stringify`.
- Any other type is converted via `String()`.

If the resolved `inputData` is `undefined` or `null` after this resolution, the node throws.

---

## Inputs & Outputs

### Inputs

Optional workflow input `data` — used as the replacement target if it's an array, or as a fallback when the config `data` field is empty.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| (root) | `string` | The input string with all (or the first, without the `g` flag) matches of `pattern` replaced by `replacement`. Unchanged (equal to the stringified input) if no matches are found. |

With the `g` flag (default), **all** matches are replaced. Without `g`, only the **first** match is replaced.

---

## Output Example

### Global Replace (default)

Pattern: `\d+`, Replacement: `#`, Input: `"Order 42 shipped, item 7 of 10"`

```text
"Order # shipped, item # of #"
```

### Capture Group Reference

Pattern: `(\d{4})-(\d{2})-(\d{2})`, Replacement: `$3/$2/$1`, Input: `"Date: 2026-08-18"`

```text
"Date: 18/08/2026"
```

### No Matches

Pattern: `\d+`, Replacement: `#`, Input: `"no numbers here"`

```text
"no numbers here"
```

### Non-Global Replace (First Match Only)

Pattern: `\d+`, Flags: `""`, Replacement: `#`, Input: `"42 and 7 and 10"`

```text
"# and 7 and 10"
```

---

## Configuration Examples

### Mask All Digits

```json
{
  "pattern": "\\d",
  "replacement": "*",
  "data": "Call 555-1234"
}
```

### Collapse Whitespace

```json
{
  "pattern": "\\s+",
  "replacement": " "
}
```

### Reformat a Date with Capture Groups

```json
{
  "pattern": "(\\d{4})-(\\d{2})-(\\d{2})",
  "replacement": "$3/$2/$1"
}
```

### Case-Insensitive Word Replacement

```json
{
  "pattern": "error",
  "flags": "gi",
  "replacement": "ISSUE",
  "data": "Error: failed. error: retry."
}
```

---

## Workflow Integration

### Sample Workflow: Regex Replace → Function

```json
{
  "nodes": [
    {
      "id": "mask-phone",
      "type": "regex-replace",
      "config": {
        "pattern": "\\d",
        "replacement": "*"
      }
    },
    {
      "id": "log-masked",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: HTTP Request → Regex Replace → LLM

```json
{
  "nodes": [
    {
      "id": "fetch-page",
      "type": "http-request"
    },
    {
      "id": "strip-html-tags",
      "type": "regex-replace",
      "config": {
        "pattern": "<[^>]+>",
        "replacement": ""
      }
    },
    {
      "id": "summarize",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: Regex Match → Regex Replace (find then clean)

```json
{
  "nodes": [
    {
      "id": "extract-numbers",
      "type": "regex-match",
      "config": {
        "pattern": "\\d+"
      }
    },
    {
      "id": "clean-text",
      "type": "regex-replace",
      "config": {
        "pattern": "\\d+",
        "replacement": "[REDACTED]"
      }
    }
  ]
}
```

### Common Patterns

- HTTP Request → Regex Replace (strip HTML) → LLM — clean scraped text before summarization
- Regex Replace (mask PII) → Database — redact sensitive data before storage
- Function (assemble text) → Regex Replace (normalize whitespace) → Notification — clean formatting before display

---

## Error Handling

### Missing Data

```text
Data is required for regex replace
```

Raised when the resolved `inputData` (from config or workflow input) is `undefined` or `null`.

### Missing Pattern

```text
Pattern is required for regex replace
```

Raised when `pattern` is empty.

### Invalid Regex / Replace Failure

```text
Regex replace failed: <error message>
```

Raised when `new RegExp(pattern, flags)` throws (invalid pattern syntax or invalid flag combination), or if `replace()` itself throws.

---

## Troubleshooting

### "Data is required for regex replace"

**Cause**

Neither the config `data` field nor the workflow's input data provided a usable value (both `undefined`/`null`, and the input wasn't an array).

**Solution**

Set the `data` config field explicitly, or ensure the upstream node passes a non-null value as input.

---

### "Pattern is required for regex replace"

**Cause**

`pattern` was left empty.

**Solution**

Provide a valid regex pattern string.

---

### "Regex replace failed: Invalid regular expression: ..."

**Cause**

The `pattern` or `flags` string is not valid JavaScript regex syntax.

**Solution**

Test the pattern in a JavaScript regex validator before using it, and confirm `flags` only contains valid flag characters (`g`, `i`, `m`, `s`, `u`, `y`, `d`).

---

### Only the First Match Was Replaced

**Cause**

`flags` was set without the `g` character (e.g. `"i"` alone), so `replace()` only replaces the first occurrence.

**Solution**

Include `g` in `flags` (e.g. `"gi"`) to replace all matches, or omit `flags` entirely to use the default `"g"`.

---

### `$1`, `$2` Appear Literally in the Output Instead of Being Substituted

**Cause**

`pattern` does not actually contain capture groups (parentheses) at the positions referenced, or the groups didn't match for a given occurrence.

**Solution**

Verify `pattern` includes parenthesized capture groups in the same order referenced by `$1`, `$2`, etc. in `replacement`.

---

### Replacing Against an Object Produces Unexpected Results

**Cause**

When the resolved input is an object, it is converted via `JSON.stringify` before replacement — so the pattern is applied to the JSON text representation (including quotes, braces, key names), not the object's individual field values.

**Solution**

If you need to replace within a specific field's value, extract that field into a string (e.g. with a `Function` node) before this node, rather than passing the whole object.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally using JavaScript's native `RegExp` engine.

Since `pattern` can be arbitrary user/workflow-supplied text compiled directly into a `RegExp`, a maliciously crafted pattern (e.g. catastrophic backtracking) could cause excessive CPU usage on large inputs — validate or restrict pattern sources if they come from untrusted input.

---

## Notes

The node returns a plain string with no additional metadata (no match count, no indication of whether any replacement actually occurred).

The node does not:

- Report how many replacements were made
- Support a replacer function (only a static replacement string with `$n` references)
- Validate the pattern for catastrophic backtracking risk before executing it
- Preserve the original data type — the output is always a string, even if the input was an object or array


---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release |
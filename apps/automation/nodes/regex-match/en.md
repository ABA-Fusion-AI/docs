---
node_id: "regex-match"
title: "Regex Match"
description: "Match string against regex pattern."
category: "Utility / Text Processing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- regex
- regular-expressions
- pattern-matching
- text-processing
- string-matching
- utility

related_nodes: []

---

# Regex Match

> **Category:** utility-nodes | **Type:** Action Node

Match a string against a **regular expression pattern** and return all matches.

The **Regex Match** node builds a JavaScript `RegExp` from a configured pattern and flags, applies it to input data (from config or the workflow input), and returns the array of matches.

### Supported Features

- Match any string against a custom regex pattern
- Configurable regex flags (e.g. `g`, `i`, `m`)
- Accepts input data from either node configuration or upstream workflow data
- Automatic array pass-through: if the upstream input is already an array, it is used directly as the match target
- Automatic stringification of non-string, non-array input (objects via `JSON.stringify`, other types via `String()`)
- Defaults to global matching (`g` flag) when no flags are specified

### Use Cases

- Extract all email addresses, URLs, or hashtags from a block of text
- Validate that a string contains an expected pattern before continuing a workflow
- Pull structured tokens (IDs, codes, dates) out of unstructured text
- Filter or route workflow data based on whether a pattern is present (via `If`, checking for a non-empty result)
- Parse log lines or semi-structured text into discrete matches for further processing

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `data` | `string` | ❌ No | — | String to match against. Used when set and non-empty; otherwise falls back to the node's input data. |
| `pattern` | `string` | ✅ Yes | — | Regular expression pattern (without delimiters), e.g. `\\d+` or `[A-Z]{2,}`. |
| `flags` | `string` | ❌ No | `"g"` | Regex flags, e.g. `gi`, `m`. Defaults to `"g"` (global) if omitted. |

---

## Input Resolution

The string matched against `pattern` is resolved in this order:

1. **If the workflow input `data` is an array** — it is used directly as `inputData`, regardless of the `data` config field.
2. **Else if the config field `data` is set and non-empty** (`!== undefined`, `!== null`, `!== ""`) — the config value is used.
3. **Else** — the workflow's input `data` is used as-is.

The resolved `inputData` is then converted to a string for matching:
- A `string` is used as-is.
- An `object` (including an array not caught by step 1's array-passthrough — see note below) is converted via `JSON.stringify`.
- Any other type is converted via `String()`.

If the resolved `inputData` is `undefined` or `null` after this resolution, the node throws.

---

## Inputs & Outputs

### Inputs

Optional workflow input `data` — used as the match target if it's an array, or as a fallback when the config `data` field is empty.

### Outputs

The node returns the raw result of `string.match(regex)`:

| Output | Type | Description |
| ------ | ---- | ----------- |
| (root) | `string[]` | Array of matched substrings. Empty array `[]` if no matches are found (never `null`). |

With the `g` flag (default), `match()` returns all non-overlapping matches without capture groups. Without the `g` flag, `match()` returns a single match with capture groups and additional metadata (`index`, `input`, `groups`) — but the node's return type is still typed as an array in practice.

---

## Output Example

### Global Match (default)

Pattern: `\d+`, Input: `"Order 42 shipped, item 7 of 10"`

```json
["42", "7", "10"]
```

### No Matches

Pattern: `\d+`, Input: `"no numbers here"`

```json
[]
```

### Case-Insensitive Match

Pattern: `error`, Flags: `gi`, Input: `"Error: failed. error: retry."`

```json
["Error", "error"]
```

---

## Configuration Examples

### Extract All Numbers

```json
{
  "pattern": "\\d+",
  "data": "Order 42 shipped, item 7 of 10"
}
```

### Extract Email Addresses

```json
{
  "pattern": "[\\w.-]+@[\\w.-]+\\.\\w+",
  "flags": "g"
}
```

### Case-Insensitive Word Match

```json
{
  "pattern": "error",
  "flags": "gi",
  "data": "Error: failed. error: retry."
}
```

### Using Upstream Workflow Data (No `data` Config)

```json
{
  "pattern": "#[a-zA-Z0-9_]+"
}
```

---

## Workflow Integration

### Sample Workflow: Regex Match → Function

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
      "id": "process-numbers",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: HTTP Request → Regex Match → If

```json
{
  "nodes": [
    {
      "id": "fetch-page",
      "type": "http-request"
    },
    {
      "id": "extract-hashtags",
      "type": "regex-match",
      "config": {
        "pattern": "#[a-zA-Z0-9_]+"
      }
    },
    {
      "id": "check-found",
      "type": "if"
    }
  ]
}
```

### Common Patterns

- HTTP Request → Regex Match → Function — scrape and parse tokens from fetched text
- Regex Match → If (check `length > 0`) → Notification — alert when a pattern is/isn't found
- Function (assemble text) → Regex Match → Database — extract and store structured tokens

---

## Error Handling

### Missing Data

```text
Data is required for regex match
```

Raised when the resolved `inputData` (from config or workflow input) is `undefined` or `null`.

### Missing Pattern

```text
Pattern is required for regex match
```

Raised when `pattern` is empty.

### Invalid Regex / Match Failure

```text
Regex match failed: <error message>
```

Raised when `new RegExp(pattern, flags)` throws (invalid pattern syntax or invalid flag combination), or if `match()` itself throws.

---

## Troubleshooting

### "Data is required for regex match"

**Cause**

Neither the config `data` field nor the workflow's input data provided a usable value (both `undefined`/`null`, and the input wasn't an array).

**Solution**

Set the `data` config field explicitly, or ensure the upstream node passes a non-null value as input.

---

### "Pattern is required for regex match"

**Cause**

`pattern` was left empty.

**Solution**

Provide a valid regex pattern string.

---

### "Regex match failed: Invalid regular expression: ..."

**Cause**

The `pattern` or `flags` string is not valid JavaScript regex syntax — e.g. unbalanced parentheses/brackets, or an unsupported flag character.

**Solution**

Test the pattern in a JavaScript regex validator before using it, and confirm `flags` only contains valid flag characters (`g`, `i`, `m`, `s`, `u`, `y`, `d`).

---

### Result is Always `[]` Even Though the Pattern Looks Right

**Cause**

Either the input string doesn't actually contain a match, or the pattern was written with regex delimiters (`/pattern/flags`) included by mistake — `pattern` should be the raw pattern text only, with `flags` supplied separately.

**Solution**

Remove any leading/trailing `/` characters from `pattern`, and move flag characters into the `flags` field instead.

---

### Matching Against an Object Produces Unexpected Results

**Cause**

When the resolved input is an object (not caught by the array-passthrough), it is converted via `JSON.stringify` before matching — so the pattern is applied to the JSON text representation, including quotes, braces, and key names, not the object's "meaning".

**Solution**

If you need to match against a specific field's value, extract that field into a string (e.g. with a `Function` node) before this node, rather than passing the whole object.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally using JavaScript's native `RegExp` engine.

Since `pattern` can be arbitrary user/workflow-supplied text compiled directly into a `RegExp`, a maliciously crafted pattern (e.g. catastrophic backtracking) could cause excessive CPU usage on large inputs — validate or restrict pattern sources if they come from untrusted input.

---

## Notes

The node returns the raw `String.prototype.match()` result with no additional post-processing (no deduplication, no capture-group extraction beyond what `match()` itself provides).

The node does not:

- Support `matchAll()` semantics (capture groups per match) — only the flag-dependent behavior of `match()`
- Deduplicate repeated matches
- Support replacing or testing (only matching) — no `replace()` or `test()` equivalent operation
- Validate the pattern for catastrophic backtracking risk before executing it

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release |
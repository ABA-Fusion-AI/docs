---
node_id: "prefix-suffix"
title: "Prefix & Suffix"
description: "Add prefix and/or suffix to a string input."
category: "Utility / Text Processing"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:

- text-processing
- string-manipulation
- prefix
- suffix
- concatenation
- utility

related_nodes:
- function
- regex-replace

---

# Prefix & Suffix

> **Category:** utility-nodes | **Type:** Action Node

Add a **prefix and/or suffix** to a string input.

The **Prefix & Suffix** node concatenates an optional prefix and suffix around input data (from config or the workflow input), converting non-string input to a string first.

### Supported Features

- Prepend a configurable prefix string
- Append a configurable suffix string
- Both prefix and suffix are optional and independent — either can be used alone
- Accepts input data from either node configuration or upstream workflow data
- Automatic stringification of non-string input (objects via `JSON.stringify`, other types via `String()`)

### Use Cases

- Wrap a value in delimiters or markup (e.g. quotes, brackets, HTML tags)
- Build a formatted label or filename from a raw value (e.g. `"report_" + date + ".pdf"`)
- Add a consistent header/footer string to generated text
- Construct a URL or path by combining a base and a dynamic segment
- Annotate values with units or labels (e.g. `"$" + amount`, `count + " items"`)

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `data` | `string` | ❌ No | — | String to wrap. Used when set and non-empty; otherwise falls back to the node's input data. |
| `prefix` | `string` | ❌ No | `""` | String to prepend to the input. |
| `suffix` | `string` | ❌ No | `""` | String to append to the input. |

---

## Input Resolution

The string that `prefix`/`suffix` are applied to is resolved in this order:

1. **If the config field `data` is set and non-empty** (`!== undefined`, `!== null`, `!== ""`) — the config value is used.
2. **Else** — the workflow's input `data` is used as-is (including if it's an array — there is no array pass-through special case in this node, unlike [Regex Match](./regex-match.md)/[Regex Replace](./regex-replace.md)).

The resolved `inputData` is then converted to a string:
- A `string` is used as-is.
- An `object` (including an array) is converted via `JSON.stringify`.
- Any other type is converted via `String()`.

If the resolved `inputData` is `undefined` or `null` after this resolution, the node throws.

---

## Inputs & Outputs

### Inputs

Optional workflow input `data` — used as the base string when the config `data` field is empty.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| (root) | `string` | The concatenation of `prefix` + stringified input + `suffix`. |

---

## Output Example

### Prefix Only

Prefix: `"$"`, Input: `42`

```text
"$42"
```

### Suffix Only

Suffix: `" items"`, Input: `7`

```text
"7 items"
```

### Both Prefix and Suffix

Prefix: `"["`, Suffix: `"]"`, Input: `"active"`

```text
"[active]"
```

### Object Input

Prefix: `"data: "`, Input: `{ "id": 1, "name": "test" }`

```text
"data: {\"id\":1,\"name\":\"test\"}"
```

---

## Configuration Examples

### Wrap in Brackets

```json
{
  "prefix": "[",
  "suffix": "]",
  "data": "active"
}
```

### Build a Filename

```json
{
  "prefix": "report_",
  "suffix": ".pdf"
}
```

### Add a Currency Symbol

```json
{
  "prefix": "$",
  "data": "42.00"
}
```

### Suffix Only

```json
{
  "suffix": " items remaining"
}
```

---

## Workflow Integration

### Sample Workflow: Prefix & Suffix → Function

```json
{
  "nodes": [
    {
      "id": "build-filename",
      "type": "prefix-suffix",
      "config": {
        "prefix": "report_",
        "suffix": ".pdf"
      }
    },
    {
      "id": "save-file",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Function → Prefix & Suffix → Notification

```json
{
  "nodes": [
    {
      "id": "compute-total",
      "type": "function"
    },
    {
      "id": "format-currency",
      "type": "prefix-suffix",
      "config": {
        "prefix": "$"
      }
    },
    {
      "id": "notify-total",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Regex Replace → Prefix & Suffix

```json
{
  "nodes": [
    {
      "id": "clean-text",
      "type": "regex-replace",
      "config": {
        "pattern": "\\s+",
        "replacement": " "
      }
    },
    {
      "id": "wrap-quote",
      "type": "prefix-suffix",
      "config": {
        "prefix": "\"",
        "suffix": "\""
      }
    }
  ]
}
```

### Common Patterns

- Function (compute value) → Prefix & Suffix (add unit/currency) → Notification — formatted alerts
- Prefix & Suffix (build filename) → Database/File Storage — consistent naming
- Regex Replace (clean) → Prefix & Suffix (wrap) → Function — text formatting pipeline

---

## Error Handling

### Missing Data

```text
Data is required for prefix/suffix operation
```

Raised when the resolved `inputData` (from config or workflow input) is `undefined` or `null`.

---

## Troubleshooting

### "Data is required for prefix/suffix operation"

**Cause**

Neither the config `data` field nor the workflow's input data provided a usable value (both `undefined` or `null`).

**Solution**

Set the `data` config field explicitly, or ensure the upstream node passes a non-null value as input.

---

### Output is Just `prefix` + `suffix` with Nothing in Between

**Cause**

The resolved input string is empty (`""`) — note that an empty string does **not** trigger the "Data is required" error, since the check is only for `undefined`/`null`.

**Solution**

Verify the upstream data or `data` config field actually contains the expected non-empty value.

---

### Object Input Produces a JSON String Instead of a Readable Value

**Cause**

The resolved input is an object (not a string or primitive), so it is converted via `JSON.stringify` before concatenation — this includes quotes, braces, and key names.

**Solution**

Extract the specific field you want to wrap (e.g. with a `Function` node) before this node, rather than passing the whole object.

---

### `prefix`/`suffix` Config Not Taking Effect

**Cause**

`prefix` and `suffix` both default to `""` — if left blank, no visible change is applied, which can look like the node isn't working.

**Solution**

Confirm `prefix` and/or `suffix` are set to non-empty values in the node configuration.

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally through simple string concatenation.

---

## Notes

The node always returns a **string**, even when the input was a number, boolean, or object — there is no type-preserving mode.

The node does not:

- Trim or otherwise sanitize the input string before concatenation
- Support inserting the prefix/suffix conditionally (they are always applied when non-empty)
- Support array pass-through the way [Regex Match](./regex-match.md) and [Regex Replace](./regex-replace.md) do — an array input here is stringified via `JSON.stringify` like any other object
- Support multiple prefix/suffix pairs in a single call

---


## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-19 | Initial release |
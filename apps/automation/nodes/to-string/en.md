---
node_id: "to-string"
title: "To String"
description: "Convert value to string."
category: "Utility / Type Conversion"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- type-conversion
- string
- stringify
- utility
- data-transformation

related_nodes:
- function
- prefix-suffix

---

# To String

> **Category:** utility-nodes | **Type:** Action Node

Convert any input value to a **string**.

The **To String** node takes a value — from node configuration or upstream workflow data — and converts it to a plain string, handling primitives and objects appropriately.

### Supported Features

- Convert strings (pass-through), numbers, booleans, and objects to string form
- Accepts input data from either node configuration or upstream workflow data
- Automatic array pass-through: if the upstream input is already an array, it is used directly as the value to convert (then JSON-stringified, since arrays are objects)
- Type-aware conversion: primitives use `String()`, objects/arrays use `JSON.stringify()`

### Use Cases

- Normalize mixed-type workflow data into a consistent string format before further text processing
- Convert a number or boolean into a display-ready string
- Serialize an object or array into a JSON string for storage, logging, or transmission
- Prepare a value for a downstream node that expects string input (e.g. [Regex Match](./regex-match.md), [Prefix & Suffix](./prefix-suffix.md))

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `data` | `string` | ❌ No | — | Value to convert. Used when set and non-empty; otherwise falls back to the node's input data. |

Despite the schema typing `data` as a `string`, in practice the *effective* input value can be of any type when sourced from the workflow's input data (see [Input Resolution](#input-resolution)) — the config field itself is string-typed, but the fallback input is not type-restricted.

---

## Input Resolution

The value converted to a string is resolved in this order:

1. **If the workflow input `data` is an array** — it is used directly as `inputData`, regardless of the `data` config field.
2. **Else if the config field `data` is set and non-empty** (`!== undefined`, `!== null`, `!== ""`) — the config value is used.
3. **Else** — the workflow's input `data` is used as-is.

If the resolved `inputData` is `undefined` or `null` after this resolution, the node throws.

---

## Conversion Logic

| Input Type | Conversion | Example |
| ---------- | ---------- | ------- |
| `string` | Returned as-is (no-op). | `"hello"` → `"hello"` |
| `number` | `String(value)` | `42` → `"42"` |
| `boolean` | `String(value)` | `true` → `"true"` |
| `object` (including arrays) | `JSON.stringify(value)` | `{ "a": 1 }` → `"{\"a\":1}"` |
| Anything else (fallback) | `String(value)` | — |

---

## Inputs & Outputs

### Inputs

Optional workflow input `data` — used as the value to convert if it's an array, or as a fallback when the config `data` field is empty.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| (root) | `string` | The stringified value. |

---

## Output Example

### Number Input

```text
42
```
→
```text
"42"
```

### Boolean Input

```text
true
```
→
```text
"true"
```

### Object Input

```json
{ "id": 1, "name": "test" }
```
→
```text
"{\"id\":1,\"name\":\"test\"}"
```

### Array Input

```json
[1, 2, 3]
```
→
```text
"[1,2,3]"
```

### String Input (No-Op)

```text
"already a string"
```
→
```text
"already a string"
```

---

## Configuration Examples

### Convert Config Value

```json
{
  "data": "42"
}
```

### Convert Upstream Data (No `data` Config)

```json
{}
```

---

## Workflow Integration

### Sample Workflow: Function (produce object) → To String → Database

```json
{
  "nodes": [
    {
      "id": "build-record",
      "type": "function"
    },
    {
      "id": "stringify-record",
      "type": "to-string"
    },
    {
      "id": "log-record",
      "type": "database"
    }
  ]
}
```

### Sample Workflow: To String → Prefix & Suffix

```json
{
  "nodes": [
    {
      "id": "stringify-value",
      "type": "to-string"
    },
    {
      "id": "wrap-value",
      "type": "prefix-suffix",
      "config": {
        "prefix": "Value: "
      }
    }
  ]
}
```

### Sample Workflow: To String → Regex Match

```json
{
  "nodes": [
    {
      "id": "stringify-data",
      "type": "to-string"
    },
    {
      "id": "extract-pattern",
      "type": "regex-match",
      "config": {
        "pattern": "\\d+"
      }
    }
  ]
}
```

### Common Patterns

- Function (compute a number/boolean/object) → To String → Notification — ensure a consistent string message body
- To String → Prefix & Suffix — build a labeled, formatted string from any input type
- To String → Regex Match/Regex Replace — normalize input type before text-pattern operations

---

## Error Handling

### Missing Data

```text
Data is required for string conversion
```

Raised when the resolved `inputData` (from config or workflow input) is `undefined` or `null`.

---

## Troubleshooting

### "Data is required for string conversion"

**Cause**

Neither the config `data` field nor the workflow's input data provided a usable value (both `undefined` or `null`).

**Solution**

Set the `data` config field explicitly, or ensure the upstream node passes a non-null value as input.

---

### Object/Array Output Looks Like Unreadable JSON

**Cause**

Objects and arrays are converted via `JSON.stringify`, which produces compact, escaped JSON text — this is expected and is not a readable/pretty-printed format.

**Solution**

If a human-readable representation is needed instead of JSON, use a `Function` node to format the object manually rather than relying on this node's default `JSON.stringify` behavior.

---

### Empty String Input Triggers the Fallback to Workflow Data

**Cause**

The config field check is `configData !== ""`, so an explicitly empty string in `data` is treated the same as "not set" and the node falls through to the workflow's input data instead.

**Solution**

If you specifically want to convert an empty string, ensure it arrives via the workflow's input data rather than the `data` config field, since an empty string there is indistinguishable from "unset."

---

## Security

The node performs no outbound HTTP requests and does not access external services.

All computation is performed locally through native JavaScript type coercion and `JSON.stringify`.

---

## Notes

The node always returns a plain **string**, with no formatting options (no pretty-printing for JSON, no locale-aware number formatting, no custom boolean labels).

The node does not:

- Pretty-print JSON output (always compact, single-line JSON for objects/arrays)
- Support custom formatting (e.g. number precision, date formatting)
- Handle circular references in objects — `JSON.stringify` will throw natively if the input object is circular, and this error is not caught or wrapped by the node
- Distinguish between an intentionally empty string and an unset `data` config field

---

## Related

- [Function](./function.md) – Custom or formatted string conversion beyond this node's default behavior
- [Prefix & Suffix](./prefix-suffix.md) – Wrap the converted string with additional text

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
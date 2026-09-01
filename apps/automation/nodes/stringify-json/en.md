---
node_id: "stringify-json"
title: "Stringify JSON"
description: "Convert data to a JSON string with optional indentation"
category: "Data Transformation (ETL)"
subcategory: "Parsing & Serialization"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - utility
  - json
  - serialization
  - stringify
  - data
related_nodes:
  - parse-json
  - serialize-canonical-json
  - function
  - log
---

<!-- SECTION: header -->

# Stringify JSON

> **Category:** Data Transformation (ETL) | **Subcategory:** Parsing & Serialization | **Type:** Action Node

Convert incoming data or configured text into a JSON string using standard JSON serialization with optional indentation.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Stringify JSON** node converts data into a JSON string using standard JSON serialization. It can stringify incoming workflow data or a configured string value and supports optional indentation through the `space` parameter.

The node is useful when structured workflow data must be converted into a JSON-formatted string before logging, storage, transmission, or further processing.

### Key Features

- **JSON Serialization:** Convert supported values using standard JSON stringification
- **Configured Data:** Stringify a value entered directly in the node configuration
- **Incoming Data:** Stringify data received from a previous node
- **Array Support:** Stringify incoming arrays directly
- **Optional Indentation:** Configure formatting using a number or string
- **Input Priority:** Select the source according to the node's input resolution rules
- **Error Handling:** Raise a descriptive error when JSON stringification fails

### Use Cases

- Convert workflow objects to JSON strings
- Serialize arrays for downstream processing
- Prepare JSON-formatted text for APIs or storage
- Format JSON data for logging
- Convert configured text into a valid JSON string representation
- Apply indentation to structured input data

### Input Resolution

The node determines the value to stringify using the following priority:

1. If the incoming workflow data is an array, the incoming array is used.
2. Otherwise, if the configured `data` parameter contains a non-empty value, the configured value is used.
3. Otherwise, the incoming workflow data is used.

This means an incoming array has priority over configured `data`, while configured `data` has priority over non-array incoming data.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | ❌ No | — | String value to serialize. Used when the incoming data is not an array and this field is not empty |
| `space` | `number` or `string` | ❌ No | — | Optional indentation value passed to JSON serialization |

### Data

The `data` parameter accepts a string.

For example:

```text
hello world
```

is serialized as:

```text
"hello world"
```

If `data` contains text that looks like JSON, the node does not parse it into an object before serialization.

For example:

```text
{"name":"Hamza","age":30}
```

is treated as a string and serialized as a JSON string representation rather than as a JSON object.

Conceptually, the resulting value corresponds to:

```text
"{\"name\":\"Hamza\",\"age\":30}"
```

To stringify an actual object or array, provide structured data through the node input instead of entering JSON text in the `data` parameter.

### Space

The `space` parameter controls indentation during JSON serialization.

It accepts either:

- a number
- a string

Example with a number:

```text
2
```

For structured data, this produces formatted JSON using indentation.

Example input:

```json
{
  "name": "John",
  "active": true
}
```

Serialized with `space: 2`:

```json
{
  "name": "John",
  "active": true
}
```

The returned value is still a JSON string even when the Log node visually displays it as formatted JSON text.

### String Indentation

The `space` parameter can also be configured as a string.

For example:

```text
--
```

When structured input contains nested values, the configured string can be used as the indentation sequence during serialization.

### Empty Data

If `data` is empty, the node falls back to incoming workflow data unless that incoming data is an array, in which case the array already has priority.

### Undefined Input

If the selected input value is `undefined`, the node returns the literal string:

```text
undefined
```

instead of calling `JSON.stringify()` on the value.

### UI Note

The `space` parameter is optional in the node schema.

Depending on the current editor control and selected parameter type, the interface may require a value to be entered when the `space` field is displayed as a numeric input.

This is an editor behavior; the underlying node schema defines `space` as optional.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Data received from the previous workflow node |

The incoming value can be an object, array, string, number, boolean, or another JSON-serializable value.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` | JSON string produced from the selected input value |
| `error` | — | Execution error when JSON stringification fails |

### String Input

Input:

```text
hello world
```

Output:

```text
"hello world"
```

### Object Input

Input:

```json
{
  "source": "function",
  "value": 123
}
```

Output:

```text
{"source":"function","value":123}
```

With indentation configured, the returned string can contain line breaks and indentation.

### Array Input

Input:

```json
[
  "apple",
  "banana",
  "orange"
]
```

Output:

```text
["apple","banana","orange"]
```

### Undefined Output

When the selected value is `undefined`, the output is:

```text
undefined
```

### Error Output

If `JSON.stringify()` throws an error, the node raises an error using the following format:

```text
JSON stringification failed: <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic Example: Stringify Text

**Configuration:**

```text
Data: hello world
Space: 0
```

**Output:**

```text
"hello world"
```

The configured string is converted into its JSON string representation.

---

### Example: Configured JSON-Like Text

**Configuration:**

```text
Data: {"name":"Hamza","age":30}
Space: 2
```

The `data` parameter is defined as a string, so the value is not parsed into an object.

The resulting JSON string corresponds to:

```text
"{\"name\":\"Hamza\",\"age\":30}"
```

---

### Example: Incoming Object

A previous node returns:

```json
{
  "source": "function",
  "value": 123
}
```

Configure:

```text
Data: empty
Space: 2
```

The node uses the incoming object because no configured `data` value is available.

The serialized content represents:

```json
{
  "source": "function",
  "value": 123
}
```

---

### Example: Incoming Array

A previous node returns:

```json
[
  "apple",
  "banana",
  "orange"
]
```

The Stringify JSON node serializes the array.

Example output:

```text
["apple","banana","orange"]
```

---

### Example: Array Priority

Suppose the incoming value is:

```json
[
  "apple",
  "banana",
  "orange"
]
```

and the node configuration contains:

```text
Data: hello world
Space: 2
```

The incoming array is used instead of the configured `data`.

Output:

```text
["apple","banana","orange"]
```

This behavior occurs because incoming arrays have the highest priority in the node's input resolution logic.

---

### Example: Configured Data Priority

Suppose the incoming object is:

```json
{
  "source": "function",
  "value": 123
}
```

and the node configuration contains:

```text
Data: hello world
Space: 2
```

The configured `data` value is used because the incoming value is not an array.

Output:

```text
"hello world"
```

---

### Example: Empty Data Fallback

Suppose the incoming object is:

```json
{
  "source": "function",
  "value": 123
}
```

and the configuration contains:

```text
Data: empty
Space: 2
```

The node falls back to the incoming object.

Serialized output:

```text
{"source":"function","value":123}
```

---

### Example: Numeric Indentation

For structured incoming data, configure:

```text
Space: 2
Type: number
```

The value is passed as the indentation argument during JSON serialization.

---

### Example: String Indentation

Configure the `space` parameter as a string:

```text
Space: --
Type: string
```

The string is passed as the indentation sequence during JSON serialization.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert data to a JSON string and inspect the result
```

### Common Patterns

- **Configured String:** Manual Trigger → Stringify JSON → Log
- **Object Serialization:** Function → Stringify JSON → Log
- **Array Serialization:** Function → Stringify JSON → Log
- **API Preparation:** Data Source → Stringify JSON → HTTP Request
- **Logging:** Data Source → Stringify JSON → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### JSON text is returned with escaped quotes

**Cause:** The `data` configuration parameter is a string. JSON-looking text entered in this field is treated as text rather than parsed into an object.

For example:

```text
{"name":"Hamza"}
```

is serialized as a JSON string representation.

**Solution:** If you need to serialize an actual object, provide the object through the node input.

---

#### Configured Data is ignored

**Cause:** The incoming workflow data is an array.

Incoming arrays have priority over the configured `data` parameter.

**Solution:** Remove or transform the incoming array if you want the node to use configured `data`.

---

#### Incoming Object is ignored

**Cause:** The `data` parameter contains a non-empty configured value.

For non-array incoming data, configured `data` has priority.

**Solution:** Clear the `data` parameter to stringify the incoming object.

---

#### Space appears to have no effect

**Cause:** Indentation is only visually meaningful for structured objects or arrays. Stringifying a simple string does not create nested JSON formatting.

**Solution:** Test indentation using structured incoming data.

---

#### Space cannot be left empty in the editor

**Cause:** The current editor control may require a value when `space` is displayed using the numeric input type.

**Solution:** Enter a valid indentation value such as:

```text
0
```

or:

```text
2
```

The underlying node schema defines `space` as optional.

---

#### JSON stringification failed

**Cause:** The selected value cannot be serialized by `JSON.stringify()`.

**Solution:** Verify that the incoming value is compatible with JSON serialization.

The node reports the underlying error using:

```text
JSON stringification failed: <error message>
```

### Error Reference

| Error / Behavior | Cause | Solution |
|------------------|-------|----------|
| Escaped JSON text | JSON entered in `data` is treated as a string | Pass an object through the input |
| Configured data ignored | Incoming data is an array | Transform or remove the incoming array |
| Incoming object ignored | `data` contains a value | Clear `data` |
| No visible indentation | Value is a simple string | Use structured input |
| Numeric Space requires value in editor | Current UI control behavior | Enter a valid number |
| `JSON stringification failed` | Value cannot be serialized | Check the incoming data |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- **Parse JSON** - Parse JSON text into structured data
- **Serialize Canonical JSON** - Serialize data using canonical JSON formatting
- **Function** - Create or transform structured input data
- **Log** - Inspect serialized output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial documentation and validated workflow example |

<!-- /SECTION: changelog -->
---
node_id: "set-field"
title: "Set Field"
description: "Write value at a path, with optional merge mode."
category: "data-transformation-etl"
subcategory: "data-shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - set-field
  - data-shaping
  - object
  - nested-path
  - merge
  - transformation
related_nodes:
  - select-fields
  - restructure
  - extract
---

<!-- SECTION: header -->
# Set Field

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Write or merge a value at a configurable object path.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Set Field** node writes a value into an object at a specified path.

It supports:

- setting a value directly;
- creating nested object paths automatically;
- parsing JSON object or array values;
- merging object values with existing object properties.

The node accepts input data from either the configured `data` parameter or incoming workflow data.

### Key Features

- Writes values to top-level object properties.
- Supports dot-separated nested paths.
- Automatically creates missing intermediate objects.
- Supports `set` and `merge` modes.
- Parses JSON object and array values when possible.
- Preserves existing object properties during merge operations.
- Rejects non-object input data.
- Returns the transformed object.

### Processing Flow

```text
Input data
  ↓
Resolve configured or incoming data
  ↓
Parse JSON when required
  ↓
Validate object input
  ↓
Split path into segments
  ↓
Create missing nested objects
  ↓
Parse value when JSON
  ↓
Set or merge value
  ↓
Return transformed object
```

### Use Cases

- Adding fields to workflow payloads.
- Updating existing properties.
- Creating nested configuration structures.
- Enriching structured data before downstream processing.
- Merging profile or metadata objects.
- Preparing objects for API calls or persistence.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | JSON object to transform. If omitted, incoming workflow data is used. |
| `path` | `string` | Yes | — | Dot-separated path where the value will be written. |
| `value` | `string` | Yes | — | Value to write. JSON objects and arrays are parsed when possible. |
| `mode` | `string` | No | — | Write mode: `set` or `merge`. |

### Data

Provide the input object as JSON.

Example:

```json
{
  "name": "Hamza",
  "age": 25
}
```

If `data` is omitted, the node uses incoming workflow data.

The input must resolve to an object.

Arrays are not accepted as the final object for field setting.

### Path

The `path` parameter identifies the property to write.

A simple path:

```text
country
```

writes a top-level field.

A nested path:

```text
profile.location.city
```

creates or navigates through nested objects.

If intermediate properties do not exist, the node creates them automatically.

For example:

```text
profile.location.city
```

can create:

```json
{
  "profile": {
    "location": {
      "city": "Rabat"
    }
  }
}
```

### Value

The `value` parameter is normally treated as a string.

If the value starts with:

```text
{
```

or:

```text
[
```

the node attempts to parse it as JSON.

For example:

```json
{"age":25,"country":"Morocco"}
```

is stored as an object rather than a plain string.

If JSON parsing fails, the original string is preserved.

### Mode

Supported values:

```text
set
merge
```

#### Set Mode

`set` writes the provided value directly at the selected path.

Example:

```text
path: country
value: Morocco
mode: set
```

Input:

```json
{
  "name": "Hamza",
  "age": 25
}
```

Output:

```json
{
  "name": "Hamza",
  "age": 25,
  "country": "Morocco"
}
```

#### Merge Mode

`merge` performs a shallow object merge when:

- the current value at the target path is an object;
- the new parsed value is also an object;
- neither value is an array.

For example:

Input:

```json
{
  "name": "Hamza",
  "profile": {
    "age": 25,
    "city": "Rabat"
  }
}
```

Configuration:

```text
path: profile
mode: merge
```

Value:

```json
{
  "country": "Morocco",
  "city": "Sale"
}
```

Output:

```json
{
  "name": "Hamza",
  "profile": {
    "age": 25,
    "city": "Sale",
    "country": "Morocco"
  }
}
```

Existing properties are preserved unless the new object provides the same key.

### Merge Fallback

If `mode` is `merge` but either the existing target value or the new value is not an object, the node falls back to direct assignment.

This means the value is replaced rather than merged.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses:

- configured `data`, when provided;
- otherwise incoming workflow data;
- `path`;
- `value`;
- optional `mode`.

The input data must resolve to an object. Arrays are not supported.

### Output

The node returns the transformed object.

Example:

```json
{
  "name": "Hamza",
  "profile": {
    "age": 25,
    "country": "Morocco"
  }
}
```

### Data Source Priority

The node resolves data in this order:

1. incoming workflow data when it is an array;
2. configured `data` when it is present and non-empty;
3. incoming workflow data;
4. an empty object when input is `null` or `undefined`.

### JSON Parsing

Configured data is parsed from JSON when provided as a string.

The `value` parameter is also parsed when it appears to contain a JSON object or array.

### Nested Path Behavior

The node splits `path` using:

```text
.
```

Each intermediate segment is converted into an object when necessary.

If an existing intermediate value is:

- missing;
- `null`;
- not an object;
- an array;

the node replaces it with an empty object before continuing.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Set a Top-Level Field

**Configuration**

```text
data: {"name":"Hamza","age":25}
path: country
value: Morocco
mode: set
```

**Output**

```json
{
  "name": "Hamza",
  "age": 25,
  "country": "Morocco"
}
```

### Example 2: Create a Nested Path

**Configuration**

```text
data: {"name":"Hamza"}
path: profile.location.city
value: Rabat
mode: set
```

**Output**

```json
{
  "name": "Hamza",
  "profile": {
    "location": {
      "city": "Rabat"
    }
  }
}
```

### Example 3: Set a JSON Object Value

**Configuration**

```text
data: {"name":"Hamza"}
path: profile
value: {"age":25,"country":"Morocco"}
mode: set
```

**Output**

```json
{
  "name": "Hamza",
  "profile": {
    "age": 25,
    "country": "Morocco"
  }
}
```

### Example 4: Merge Object Properties

**Configuration**

```text
data: {"name":"Hamza","profile":{"age":25,"city":"Rabat"}}
path: profile
value: {"country":"Morocco","city":"Sale"}
mode: merge
```

**Output**

```json
{
  "name": "Hamza",
  "profile": {
    "age": 25,
    "city": "Sale",
    "country": "Morocco"
  }
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Set Field Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Data Is Not Valid JSON

**Cause:** Configured `data` is a string that cannot be parsed as JSON.

The node throws:

```text
Data must be a valid JSON object or object for field setting
```

**Solution:** Provide valid JSON object data.

### Data Is Not an Object

**Cause:** The resolved data is a primitive value or an array.

For example:

```json
"hello"
```

The node throws:

```text
Data must be an object for field setting
```

**Solution:** Provide an object such as:

```json
{
  "name": "Hamza"
}
```

### Path Is Missing

If `path` is empty at runtime, the node throws:

```text
Path is required for field setting
```

Provide a valid property path.

### Existing Nested Value Is Replaced

When navigating a nested path, an intermediate property that is not a plain object is replaced with an empty object.

For example, if:

```json
{
  "profile": "text"
}
```

and the path is:

```text
profile.location.city
```

the original `profile` string is replaced by an object so the nested path can be created.

### Merge Does Not Preserve the Existing Value

Merge mode only performs object merging when both values are non-array objects.

If either side is a string, number, boolean, array, or `null`, direct assignment is used instead.

### Merge Is Shallow

The merge operation uses a shallow object merge.

For example:

```json
{
  "profile": {
    "settings": {
      "theme": "dark"
    }
  }
}
```

merged with:

```json
{
  "settings": {
    "language": "en"
  }
}
```

replaces the existing `settings` object rather than recursively merging its nested properties.

### Invalid JSON Value Is Preserved as Text

If `value` begins like JSON but parsing fails, the node keeps the original string.

This behavior prevents malformed JSON-like values from causing the operation to fail automatically.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Select Fields** — Select specific fields from structured data.
- **Restructure** — Reshape structured workflow data.
- **Extract** — Extract values from structured input.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for the Set Field node. |

<!-- /SECTION: changelog -->
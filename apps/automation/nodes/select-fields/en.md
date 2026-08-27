---
node_id: "select-fields"
title: "Select Fields"
description: "Pick specific fields from an object or array of objects with optional renaming. Supports nested paths using dot notation."
category: "data-transformation-etl"
subcategory: "data-shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - select-fields
  - data-shaping
  - field-selection
  - nested-path
  - rename
  - transformation
related_nodes:
  - set-field
  - extract
  - restructure
---

<!-- SECTION: header -->
# Select Fields

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Select specific fields from an object or array of objects, with optional renaming and nested path support.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Select Fields** node extracts selected fields from structured data.

It supports:

- selecting fields from a single object;
- selecting fields from arrays of objects;
- renaming selected fields;
- reading nested values using dot notation;
- configurable handling of missing fields.

The node accepts input data from either the configured `data` parameter or incoming workflow data.

### Key Features

- Selects specific fields from objects.
- Supports arrays of objects.
- Supports dot-separated nested paths.
- Supports optional field renaming.
- Supports configurable missing-field handling.
- Can skip missing fields.
- Can include missing fields.
- Can throw an error when a required field is missing.
- Preserves primitive array items without processing them.
- Returns only the selected fields for valid object items.

### Processing Flow

```text
Input data
  ↓
Resolve configured or incoming data
  ↓
Parse JSON when applicable
  ↓
Object or array?
  ├─ Object → Process selected fields
  └─ Array  → Process each object item
                  ↓
              Resolve nested paths
                  ↓
              Handle missing fields
                  ↓
              Apply optional rename
                  ↓
              Return selected result
```

### Use Cases

- Reducing workflow payloads to required fields.
- Renaming fields before downstream processing.
- Extracting nested values.
- Preparing data for APIs or databases.
- Processing arrays of structured records.
- Removing unnecessary fields from workflow data.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | JSON object or array to process. If omitted, incoming workflow data is used. |
| `fields` | `array<object>` | Yes | — | List of fields to select. Each item contains `from` and optional `to`. |
| `onMissing` | `string` | No | `include` | Missing-field behavior: `skip`, `include`, or `throw`. |

### Data

Provide an object or array of objects.

Example object:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

Example array:

```json
[
  {
    "name": "Hamza",
    "country": "Morocco"
  },
  {
    "name": "Sara",
    "country": "France"
  }
]
```

If `data` is omitted, incoming workflow data is used.

If `data` is provided as a JSON string beginning with `{` or `[`, the node attempts to parse it automatically.

### Fields

Each item in `fields` contains:

```text
from
to
```

`from` identifies the source field.

`to` optionally defines a new output field name.

Example:

```text
from: name
to: fullName
```

produces:

```json
{
  "fullName": "Hamza"
}
```

If `to` is omitted, the original `from` value is used as the output key.

### Nested Paths

The `from` parameter supports dot notation.

Example:

```text
profile.address.city
```

Input:

```json
{
  "profile": {
    "address": {
      "city": "Sale"
    }
  }
}
```

Configuration:

```text
from: profile.address.city
to: city
```

Output:

```json
{
  "city": "Sale"
}
```

If any segment in the nested path does not exist, the field is treated as missing.

### On Missing

Supported values:

```text
include
skip
throw
```

#### Include

When:

```text
onMissing: include
```

a missing field is added to the result with an `undefined` value internally.

Depending on serialization or display behavior, fields with `undefined` values may not appear in the final visible JSON output.

#### Skip

When:

```text
onMissing: skip
```

missing fields are omitted from the result.

Example:

Input:

```json
{
  "name": "Hamza"
}
```

Fields:

```text
name
email
```

Output:

```json
{
  "name": "Hamza"
}
```

### Throw

When:

```text
onMissing: throw
```

the node throws an error when a selected field does not exist.

Example:

```text
Required field "email" is missing from the data
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses:

- configured `data`, when provided;
- otherwise incoming workflow data;
- `fields`;
- optional `onMissing`.

### Object Input

For a single object, the node returns a new object containing only the selected fields.

Example input:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com",
  "country": "Morocco"
}
```

Selected fields:

```text
name
email
```

Output:

```json
{
  "name": "Hamza",
  "email": "hamza@example.com"
}
```

### Array Input

When input data is an array, each item is processed independently.

Example input:

```json
[
  {
    "name": "Hamza",
    "country": "Morocco"
  },
  {
    "name": "Sara",
    "country": "France"
  }
]
```

Fields:

```text
name → fullName
country → location
```

Output:

```json
[
  {
    "fullName": "Hamza",
    "location": "Morocco"
  },
  {
    "fullName": "Sara",
    "location": "France"
  }
]
```

### Non-Object Array Items

If an array contains a primitive value, `null`, or another non-object item, that item is returned unchanged.

For example:

```json
[
  {
    "name": "Hamza"
  },
  "text",
  null
]
```

the object item is processed, while `"text"` and `null` are preserved as-is.

### No Fields Configured

If `fields` is empty or unavailable at runtime, the node returns the input data unchanged.

If the input is a JSON string beginning with `{` or `[`, it attempts to parse the string before returning it.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Select Fields

**Configuration**

```text
data: {"name":"Hamza","age":25,"email":"hamza@example.com","country":"Morocco"}

fields:
  - from: name
  - from: email

onMissing: include
```

**Output**

```json
{
  "name": "Hamza",
  "email": "hamza@example.com"
}
```

### Example 2: Rename Fields

**Configuration**

```text
data: {"name":"Hamza","country":"Morocco"}

fields:
  - from: name
    to: fullName
  - from: country
    to: location
```

**Output**

```json
{
  "fullName": "Hamza",
  "location": "Morocco"
}
```

### Example 3: Select Nested Fields

**Configuration**

```text
data: {"name":"Hamza","profile":{"age":25,"address":{"city":"Sale","country":"Morocco"}}}

fields:
  - from: profile.address.city
    to: city
  - from: profile.age
    to: age
```

**Output**

```json
{
  "city": "Sale",
  "age": 25
}
```

### Example 4: Skip Missing Fields

**Configuration**

```text
data: {"name":"Hamza","age":25}

fields:
  - from: name
  - from: email

onMissing: skip
```

**Output**

```json
{
  "name": "Hamza"
}
```

### Example 5: Throw on Missing Field

**Configuration**

```text
data: {"name":"Hamza","age":25}

fields:
  - from: name
  - from: email

onMissing: throw
```

The node throws:

```text
Required field "email" is missing from the data
```

### Example 6: Process an Array

**Configuration**

```text
data: [{"name":"Hamza","age":25,"country":"Morocco"},{"name":"Sara","age":30,"country":"France"}]

fields:
  - from: name
    to: fullName
  - from: country
    to: location

onMissing: include
```

**Output**

```json
[
  {
    "fullName": "Hamza",
    "location": "Morocco"
  },
  {
    "fullName": "Sara",
    "location": "France"
  }
]
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Select Fields Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Data Is Required

If resolved input data is missing, the node throws:

```text
Data is required for field selection
```

Provide valid `data` or incoming workflow input.

### Data Is Not an Object or Array

If the resolved data is a primitive value such as:

```text
Hamza
```

the node throws:

```text
Data must be an object or array for field selection
```

Use an object or array instead.

### Required Field Is Missing

When:

```text
onMissing: throw
```

and a field cannot be resolved, the node throws:

```text
Required field "<field>" is missing from the data
```

Check the field name or nested path.

### Missing Field Is Not Visible

When:

```text
onMissing: include
```

missing fields are assigned `undefined`.

Some JSON serializers and workflow displays omit properties whose value is `undefined`.

This means the node can complete successfully even if the missing field is not visible in the displayed result.

### Nested Field Is Missing

Nested values are resolved using dot notation.

For:

```text
profile.address.city
```

every intermediate object must exist.

If `profile` or `address` is missing, the final field is treated as missing and handled according to `onMissing`.

### Primitive Values Inside Arrays

Array items that are not objects are returned unchanged.

This is expected behavior and prevents field-selection processing from failing on mixed arrays.

### Invalid JSON Data

If configured `data` begins with `{` or `[` but cannot be parsed, the node keeps the original string.

Because field selection requires an object or array, this can later result in:

```text
Data must be an object or array for field selection
```

### Empty Fields Configuration

If no fields are configured, the node returns the input data unchanged.

This is expected behavior.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Set Field** — Add or update fields in structured data.
- **Extract** — Extract values from structured input.
- **Restructure** — Reshape structured workflow data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation for the Select Fields node. |

<!-- /SECTION: changelog -->
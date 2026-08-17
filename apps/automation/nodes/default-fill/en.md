---
node_id: "default-fill"
title: "Default Fill"
description: "Fill null/undefined values with defaults."
category: "data-transformation-etl"
subcategory: "data-shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - defaults
  - data
  - fill
  - transformation
  - data-shaping
related_nodes:
  - normalize-whitespace
  - extract
  - restructure
---

<!-- SECTION: header -->
# Default Fill

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Fill missing, null, or undefined object values using configured default values.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Default Fill** node fills missing, `null`, or `undefined` object properties using configured default values.

Existing non-null values are preserved and are not overwritten.

The node accepts object data directly or parses object data provided as a string.

### Key Features

- Fills missing object properties.
- Replaces `null` values with configured defaults.
- Replaces `undefined` values with configured defaults.
- Preserves existing non-null values.
- Supports configured JSON object strings.
- Supports incoming workflow objects.
- Supports incoming object strings.
- Returns the completed object directly.

### Processing Flow

```text
Input
  ↓
Resolve configured or incoming data
  ↓
Parse string input when required
  ↓
Validate object
  ↓
Apply configured defaults
  ↓
Preserve existing values
  ↓
Return completed object
```

### Use Cases

- Filling missing user profile fields.
- Applying fallback configuration values.
- Completing partially populated objects.
- Normalizing incoming workflow data.
- Preparing objects for downstream processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | No | — | Object data provided as a JSON or object-expression string. If empty, incoming workflow data is used. |
| `defaults` | `object` | Yes | — | Key-value pairs used when a field is missing, `null`, or `undefined`. |

### Data

Provide an object as a JSON string.

Example:

```json
{"name":"Hamza","country":null}
```

The node parses the configured string before applying defaults.

### Defaults

Configure the values to use for missing, `null`, or `undefined` fields.

Example:

```json
{
  "country": "Morocco",
  "role": "Developer"
}
```

### Input Priority

Input is resolved in this order:

1. If configured `data` contains a non-empty value, the configured value is parsed and used.
2. Otherwise, incoming workflow data is used.
3. If incoming workflow data is a string, the node attempts to parse it as an object.

### Fill Behavior

A default value is applied only when the existing property is:

```text
undefined
null
```

Existing values are preserved.

For example:

```json
{
  "country": "France"
}
```

with:

```json
{
  "country": "Morocco"
}
```

remains:

```json
{
  "country": "France"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node accepts either:

- configured object data from `data`;
- incoming workflow object data;
- incoming workflow data provided as an object string.

### Input Parsing

When input is a string, the node first attempts to parse it as JSON.

If JSON parsing fails, the node attempts to parse the value using the secure expression evaluator.

The parsed result must be an object.

Arrays and primitive values are not accepted as valid object input.

### Output

The node returns the completed object directly.

Example input:

```json
{
  "name": "Hamza",
  "country": null
}
```

Defaults:

```json
{
  "country": "Morocco",
  "role": "Developer"
}
```

Output:

```json
{
  "name": "Hamza",
  "country": "Morocco",
  "role": "Developer"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fill Null and Missing Values

**Input**

```json
{
  "name": "Hamza",
  "country": null
}
```

**Defaults**

```json
{
  "country": "Morocco",
  "role": "Developer"
}
```

**Output**

```json
{
  "name": "Hamza",
  "country": "Morocco",
  "role": "Developer"
}
```

### Example 2: Preserve Existing Values

**Input**

```json
{
  "name": "Hamza",
  "country": "France"
}
```

**Defaults**

```json
{
  "country": "Morocco",
  "role": "Developer"
}
```

**Output**

```json
{
  "name": "Hamza",
  "country": "France",
  "role": "Developer"
}
```

### Example 3: Apply Defaults to Incoming Data

If `data` is empty, the node uses incoming workflow data.

Existing properties are preserved and missing properties are added from `defaults`.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Default Fill Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Invalid Object String

**Cause:** The configured or incoming string could not be parsed as a valid object.

**Solution:** Provide valid JSON object data.

Example:

```json
{"name":"Hamza","country":null}
```

### Data must be a JSON object

**Cause:** The parsed value is an array or primitive value instead of an object.

**Solution:** Provide an object rather than an array, string, number, or boolean.

### Existing Value Is Not Replaced

This is expected behavior.

The node only applies defaults when a property is missing, `null`, or `undefined`.

Existing non-null values are preserved.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Normalize Whitespace** — Normalize whitespace and remove control characters.
- **Extract** — Extract values from incoming data.
- **Restructure** — Restructure object data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial documentation for the Default Fill node. |

<!-- /SECTION: changelog -->
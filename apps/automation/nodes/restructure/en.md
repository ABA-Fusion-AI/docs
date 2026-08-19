---
node_id: "restructure"
title: "Restructure"
description: "Template-based object restructuring using expression paths."
category: "Data Transformation (ETL)"
subcategory: "Data Shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - data
  - object
  - restructure
  - mapping
  - transform
  - etl
related_nodes:
  - function
  - set
  - merge
  - log
---

<!-- SECTION: header -->
# Restructure

> **Category:** Data Transformation (ETL) | **Subcategory:** Data Shaping | **Type:** Action Node

Create a new object from incoming data by defining a template whose values point to source fields or nested fields using dot-separated expression paths.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Restructure** node reshapes an object into a new structure. Each property in the template becomes a property in the output, and its value is resolved from the incoming input using a field name or a nested expression path.

### Key Features

- **Field Mapping:** Rename fields while creating the output object
- **Nested Paths:** Read values such as `user.name` or `location.city`
- **Selective Output:** Include only the fields needed by downstream nodes
- **Shape Conversion:** Convert an existing object into the structure expected by an API or database
- **Workflow Friendly:** Use with Function, HTTP, database, validation, or logging nodes

### Typical Use Cases

- Map API responses to an internal data model
- Select and rename customer or user fields
- Prepare records before storing them in a database
- Flatten or simplify nested workflow data
- Create request payloads for downstream services
- Standardize differently shaped input objects

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` or `object` | No | Incoming input | Source object to restructure. It may be provided directly or through the incoming `input` connection. |
| `template` | `object` | Yes | `{}` | Output shape. Each key is an output property and each value is a source field name or expression path. |

### Template Values

Template values are resolved against the source object:

| Template value | Meaning |
|----------------|---------|
| `firstName` | Read the top-level `firstName` property |
| `user.name` | Read the nested `name` property inside `user` |
| `location.city` | Read the nested `city` property inside `location` |

The template can rename fields by using a different output key:

```json
{
  "name": "firstName",
  "surname": "lastName",
  "userAge": "age"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `any` | Incoming source data used to resolve template paths |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Newly created object matching the configured template |
| `error` | `object` | Error details when the source or template cannot be processed |

### Example Success Output

For this input:

```json
{
  "firstName": "Meryem",
  "lastName": "ghanem",
  "age": "28"
}
```

and this template:

```json
{
  "name": "firstName",
  "surname": "lastName",
  "userAge": "age"
}
```

the node returns:

```json
{
  "name": "Meryem",
  "surname": "ghanem",
  "userAge": "28"
}
```

### Nested Path Example

Input:

```json
{
  "user": {
    "name": "Meryem",
    "email": "meryem@example.com"
  },
  "location": {
    "city": "Casablanca"
  }
}
```

Template:

```json
{
  "username": "user.name",
  "email": "user.email",
  "city": "location.city"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Rename Top-Level Fields

Create a simplified user object from flat input.

**Input:**

```json
{
  "firstName": "Meryem",
  "lastName": "ghanem",
  "age": "28"
}
```

**Template:**

```json
{
  "name": "firstName",
  "surname": "lastName",
  "userAge": "age"
}
```

**Result:**

```json
{
  "name": "Meryem",
  "surname": "ghanem",
  "userAge": "28"
}
```

---

### Example 2: Map Nested Fields

Flatten selected fields from a nested profile object.

**Template:**

```json
{
  "username": "user.name",
  "email": "user.email",
  "city": "location.city"
}
```

**Result:**

```json
{
  "username": "Meryem",
  "email": "meryem@example.com",
  "city": "Casablanca"
}
```

---

### Example 3: Build an API Payload

Select only the fields required by a downstream service.

**Template:**

```json
{
  "customer_name": "profile.name",
  "customer_email": "profile.email",
  "country": "address.country"
}
```

This produces a compact payload without copying unrelated source properties.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Restructure object data and inspect the result
```

### Common Patterns

- **Field Mapping:** Trigger → Restructure → Log
- **Nested Data Shaping:** Function → Restructure → Log
- **API Preparation:** Restructure → HTTP Request
- **Database Preparation:** Restructure → Database or Storage Node
- **Validation Flow:** Restructure → Validate → Next Action

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Output fields are missing or empty

**Cause:** A template path does not exist in the incoming object.

**Solution:** Verify the spelling and nesting of the source path, for example `user.name`.

#### The node does not use the expected source data

**Cause:** The source object was not supplied through the input connection or `data` parameter.

**Solution:** Connect a node that outputs the source object, or provide the object directly in `data`.

#### The output shape is incorrect

**Cause:** Template keys define output property names, while template values define source paths.

**Solution:** Put the desired output name on the left and the source field/path on the right.

#### Nested values cannot be resolved

**Cause:** An intermediate object in the expression path is missing or has a different shape.

**Solution:** Inspect the incoming data and ensure every segment in the path exists.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Input is required` | No source object was provided | Connect an input or configure `data` |
| `Template is required` | No output template was configured | Add a template object |
| `Invalid template` | Template is not a valid object | Use a JSON object with output-to-source mappings |
| `Path not found` | A source expression cannot be resolved | Check the field name and nested path |

<!-- /SECTION: troubleshooting -->

---

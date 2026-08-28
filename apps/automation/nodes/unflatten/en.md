---
node_id: "unflatten"
title: "Unflatten"
description: "Convert keys with separators (., _, -, /, |) back to nested objects."
category: "Data Transformation (ETL)"
subcategory: "Data Shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-28"
author: "Fusion Team"
tags:
  - unflatten
  - flatten
  - data-shaping
  - transformation
  - json
  - nested-objects
  - object-manipulation
related_nodes:
  - flatten
  - map-keys
  - select-key
  - function
  - log
---

<!-- SECTION: header -->
# Unflatten

> **Category:** Data Transformation (ETL) | **Subcategory:** Data Shaping | **Type:** Action Node

Transform flat objects with delimited compound keys (such as `user.name` or `server_port`) back into deep, structured, multi-level nested JSON hierarchies.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Unflatten** node reverses key flattening by splitting compound object keys according to a chosen separator (`.`, `_`, `-`, `/`, or `|`) and recursively reconstructing the corresponding nested object structure.

This node is essential when dealing with flat data stores (such as SQL tables, CSV sheets, flat environment configurations, or form submissions) that need to be reshaped into structured nested JSON payloads expected by modern APIs or databases.

### Key Features

- **Multiple Delimiter Support:** Supports dot (`.`), underscore (`_`), hyphen (`-`), forward slash (`/`), and pipe (`|`) delimiters.
- **Dynamic Input Fallback:** Accepts flat data directly from node configuration or dynamically from incoming workflow payloads (`input`).
- **Automatic JSON String Parsing:** If a JSON string is passed, it is automatically parsed into an object before unflattening.
- **Built-in Prototype Pollution Guard:** Automatically ignores and strips unsafe keys (`__proto__`, `constructor`, `prototype`) across all path depths to maintain system security.
- **Deep Nesting Hierarchy:** Constructs arbitrarily deep object levels smoothly (e.g. `a.b.c.d.e` → `{ a: { b: { c: { d: { e: "..." } } } } }`).

### Use Cases

- **Database Row Normalization:** Convert flat relational database query results with prefixed columns (`customer_address_city`) into nested domain models.
- **CSV & Spreadsheet Import:** Turn flat spreadsheet rows into rich nested JSON objects before sending them to REST APIs.
- **Form & URL Route Mapping:** Reshape flat web form key-value pairs or slash-delimited route parameters (`api/v1/auth/token`) into hierarchical configs.
- **Reversing Flatten Operations:** Restore deep objects previously flattened for serialization, filtering, or temporary processing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `any` | Conditional | — | The flat object or JSON string to unflatten. If omitted or empty, the node reads from incoming workflow data. |
| `separator` | `enum` | ❌ No | `.` | The separator character used in the keys to identify nesting levels. Options: `.`, `_`, `-`, `/`, `|`. |

---

### Parameter Details

#### `data`
The target flat data to transform.
- Can be a JavaScript object, a JSON string, or an expression referencing a previous node's output (e.g., `{{input}}` or `{{Function.output}}`).
- If left empty (`""`, `null`, or `undefined`), the node automatically falls back to the incoming payload passed to its input port.

#### `separator`
Specifies the delimiter used to split key paths. Supported values:
- `.` *(Dot — Default)*: Splits keys like `user.profile.name`.
- `_` *(Underscore)*: Splits keys like `user_profile_name`.
- `-` *(Hyphen)*: Splits keys like `user-profile-name`.
- `/` *(Forward Slash)*: Splits keys like `user/profile/name`.
- `|` *(Pipe)*: Splits keys like `user|profile|name`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. Used as the flat data source when the `data` parameter is left empty. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when transformation succeeds. Contains the reconstructed nested object hierarchy. |
| `error` | `Error` | Emitted if data is missing, not an object, or contains invalid JSON string formatting. |

---

### Transformation Examples by Separator

#### 1. Dot Separator (`.`) [Default]

**Flat Input:**
```json
{
  "user.name": "Karim",
  "user.age": 28,
  "user.address.city": "Casablanca",
  "user.address.country": "Morocco"
}
```

**Unflattened Output:**
```json
{
  "user": {
    "name": "Karim",
    "age": 28,
    "address": {
      "city": "Casablanca",
      "country": "Morocco"
    }
  }
}
```

---

#### 2. Underscore Separator (`_`)

**Flat Input:**
```json
{
  "server_host": "127.0.0.1",
  "server_config_port": 8080,
  "server_config_ssl": true
}
```

**Unflattened Output:**
```json
{
  "server": {
    "host": "127.0.0.1",
    "config": {
      "port": 8080,
      "ssl": true
    }
  }
}
```

---

#### 3. Slash Separator (`/`)

**Flat Input:**
```json
{
  "api/v1/auth/token": "xyz123abc",
  "api/v1/auth/expiresIn": 3600
}
```

**Unflattened Output:**
```json
{
  "api": {
    "v1": {
      "auth": {
        "token": "xyz123abc",
        "expiresIn": 3600
      }
    }
  }
}
```

---

#### 4. Pipe Separator (`|`)

**Flat Input:**
```json
{
  "company|department|team|name": "Engineering",
  "company|location": "Rabat"
}
```

**Unflattened Output:**
```json
{
  "company": {
    "department": {
      "team": {
        "name": "Engineering"
      }
    },
    "location": "Rabat"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Direct Configuration in Node Parameters

Transform a static flat JSON object into a nested object using dot notation.

**Parameter Configuration:**

```text
Separator: .
Data:
{
  "user.name": "Karim",
  "user.age": 28,
  "user.address.city": "Casablanca",
  "user.address.country": "Morocco"
}
```

---

### Example 2: Dynamic Input from Upstream Function / Database

Pass flat query results from a database or HTTP node dynamically into the Unflatten node.

**Workflow Pattern:**

```text
Manual Trigger
  → Function (returns flat database row)
  → Unflatten (data: left empty to use input, separator: "_")
  → Log
```

---

### Example 3: Reversible Pipeline (Flatten → Process → Unflatten)

Flatten a deeply nested object for bulk filtering or searching, then unflatten it back to its original schema.

**Workflow Pattern:**

```text
Webhook Trigger
  → Flatten (separator: ".")
  → Filter Array / Map Keys (modify flat keys)
  → Unflatten (separator: ".")
  → HTTP Request (POST nested object)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Unflatten flat key-value pairs into nested objects
```

### Common Patterns

- **CSV to API Pipeline:** Read CSV / Spreadsheet → For Each Row → Unflatten (`separator: "."`) → Send to REST API.
- **SQL Reshaping:** SQL Query (prefixed columns like `order_customer_name`) → Unflatten (`separator: "_"`) → Emit structured order object.
- **Config Parser:** Read `.env` / key-value store → Unflatten → Generate nested application configuration.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `Data is required for unflatten operation`

**Cause:** Neither the `data` parameter in configuration was set nor did the upstream node emit valid data (`input` was `null` or `undefined`).

**Solution:** Provide an object in the `data` parameter or connect an upstream node that emits a non-empty object.

#### Error: `Data must be an object for unflatten operation`

**Cause:** The provided data is a primitive (number, boolean) or an array (`[]`) instead of a key-value object (`{}`).

**Solution:** Ensure the input data is a plain JavaScript object or a JSON string representing an object (e.g. `{ "a.b": 1 }`).

#### Error: `Data must be a valid JSON object or object for unflatten operation`

**Cause:** A string was passed to `data`, but it is malformed JSON and cannot be parsed.

**Solution:** Check the JSON string syntax for missing quotes, trailing commas, or invalid formatting.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Data is required for unflatten operation` | Missing data input | Provide `data` parameter or connect upstream node |
| `Data must be an object for unflatten operation` | Received Array or primitive | Pass a dictionary/object `{}` rather than `[]` |
| `Data must be a valid JSON object or object for unflatten operation` | Invalid JSON string syntax | Verify string is valid JSON before parsing |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Flatten](../flatten/en.md) — Flatten deeply nested objects into single-level delimited keys
- [Map Keys](../map-keys/en.md) — Rename or re-map object keys
- [Select Key](../select-key/en.md) — Extract specific keys from an object
- [Function](../function/en.md) — Run custom JavaScript transformations
- [Log](../log/en.md) — Inspect output structures in workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-28 | Initial release |

<!-- /SECTION: changelog -->

---
node_id: "schema-validate"
title: "Schema Validate"
description: "Production-grade validation with JSON Schema or Laravel-style rules. Supports detailed error messages, conditional validation, and custom rules."
category: "data-transformation-etl"
subcategory: "date-time"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - validation
  - schema
  - json-schema
  - laravel
  - data-validation
  - data-cleaning
related_nodes:
  - validate-phone
  - validate-url
  - html-sanitize
---

<!-- SECTION: header -->
# Schema Validate

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Validate and transform structured data using JSON Schema or Laravel-style validation rules.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Schema Validate** node validates structured data against configurable validation rules.

It supports two validation modes:

- `jsonSchema` — validates data using JSON Schema definitions.
- `laravel` — validates data using Laravel-style validation rules.

The node can return detailed validation errors and provides options for controlling error collection and data transformation.

Depending on the selected validation mode, it can also coerce compatible values to expected types, apply default values, remove additional properties, and enforce strict JSON Schema validation options.

### Key Features

- Supports JSON Schema validation.
- Supports Laravel-style validation rules.
- Returns detailed validation errors.
- Supports returning multiple validation errors.
- Supports stopping on the first validation error.
- Supports custom validation messages.
- Supports automatic type coercion in JSON Schema mode.
- Supports default values in JSON Schema mode.
- Supports removal of additional properties.
- Supports strict JSON Schema validation options.
- Returns the validated and transformed data when validation succeeds.

### Processing Flow

```text
Input data
  ↓
Select validation mode
  ↓
Load validation schema or rules
  ↓
Apply validation options
  ↓
Validate data
  ↓
Validation successful?
  ├─ Yes → Return validated data
  └─ No  → Build validation error
             ↓
           Throw error
```

### Use Cases

- Validating workflow payloads before further processing.
- Validating API request or response data.
- Enforcing required fields and data types.
- Validating email addresses and numeric constraints.
- Cleaning unexpected properties from structured data.
- Applying default values to missing properties.
- Converting compatible input values to expected types.
- Validating data using Laravel-style rules.
- Preventing invalid data from reaching downstream workflow nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `validationMode` | `string` | Yes | — | Validation mode. Supports `jsonSchema` and `laravel`. |
| `data` | `object` | Yes | — | Data to validate. |
| `schema` | `object` | Yes | — | JSON Schema definition or Laravel-style validation rules. |
| `stopOnFirstError` | `boolean` | No | `false` | Stops validation after the first detected error when enabled. |
| `returnAllErrors` | `boolean` | No | `true` | Returns multiple validation errors when supported and enabled. |
| `customMessages` | `object` | No | — | Custom validation messages. |
| `removeAdditional` | `boolean` | No | `false` | Removes additional properties during JSON Schema validation when applicable. |
| `useDefaults` | `boolean` | No | `true` | Applies default values defined by the JSON Schema. |
| `coerceTypes` | `boolean` | No | `true` | Converts compatible values to the types required by the JSON Schema. |
| `strict` | `boolean` | No | `false` | Enables strict JSON Schema validation behavior. |
| `strictNumbers` | `boolean` | No | `false` | Enables stricter handling of numeric values. |
| `strictTypes` | `boolean` | No | `false` | Enables stricter JSON Schema type validation. |

### Validation Mode

The `validationMode` parameter determines which validation syntax is used.

Supported values:

```text
jsonSchema
laravel
```

Use:

```text
jsonSchema
```

when the `schema` parameter contains a JSON Schema definition.

Use:

```text
laravel
```

when the `schema` parameter contains Laravel-style validation rules.

### Data

The `data` parameter contains the structured data to validate.

Example:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

When validation succeeds, the node returns the validated data.

Depending on the enabled JSON Schema options, the returned data may differ from the original data because values can be coerced, defaults can be inserted, and additional properties can be removed.

### Schema

The meaning of `schema` depends on `validationMode`.

#### JSON Schema

Example:

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "minLength": 2
    },
    "age": {
      "type": "integer",
      "minimum": 18
    },
    "email": {
      "type": "string",
      "format": "email"
    }
  },
  "required": [
    "name",
    "age",
    "email"
  ],
  "additionalProperties": false
}
```

This schema requires:

- `name` to be a string containing at least two characters;
- `age` to be an integer greater than or equal to `18`;
- `email` to be a valid email address;
- all three properties to be present.

#### Laravel Rules

Example:

```json
{
  "name": "required|string|min:2",
  "email": "required|email",
  "age": "required|integer|min:18"
}
```

These rules require:

- `name` to be present, be a string, and contain at least two characters;
- `email` to be present and contain a valid email address;
- `age` to be present, be an integer, and be at least `18`.

### Stop On First Error

When:

```text
stopOnFirstError: false
```

validation can continue after detecting an invalid field.

When:

```text
stopOnFirstError: true
```

the node stops validation after the first detected validation error.

For example, given invalid values for `name`, `age`, and `email`, enabling this option can result in only the first error being returned.

### Return All Errors

When enabled, the node can return multiple validation errors in the same failure.

For example:

```text
Validation failed:
The name must be at least 2 characters.;
The age must be at least 18.;
The email must be a valid email.
```

This is useful when the workflow should report multiple invalid fields at once instead of requiring repeated validation attempts.

### Custom Messages

The `customMessages` parameter can be used to configure custom validation messages where supported by the selected validation mode.

Use custom messages when workflow-specific or user-facing error text is required.

### Remove Additional

The `removeAdditional` option controls the handling of properties that are not permitted by a JSON Schema.

For example, with:

```json
{
  "additionalProperties": false
}
```

and input data:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com",
  "secret": "should-be-removed"
}
```

when:

```text
removeAdditional: true
```

the additional `secret` property is removed.

The resulting data is:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

### Use Defaults

When `useDefaults` is enabled, default values defined by the JSON Schema can be inserted into the validated data.

For example:

```json
{
  "country": {
    "type": "string",
    "default": "Morocco"
  }
}
```

If the input does not contain `country`, the returned data can contain:

```json
{
  "country": "Morocco"
}
```

### Coerce Types

When `coerceTypes` is enabled, compatible values can be converted to the type required by the JSON Schema.

For example, input:

```json
{
  "age": "25"
}
```

with a schema requiring:

```json
{
  "age": {
    "type": "integer"
  }
}
```

can be returned as:

```json
{
  "age": 25
}
```

The string value `"25"` is therefore converted to the numeric value `25`.

### Strict Options

The node exposes the following strict JSON Schema options:

```text
strict
strictNumbers
strictTypes
```

These options provide additional control over strict schema validation behavior.

They are disabled by default in the standard configuration.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `validationMode`
- `data`
- `schema`
- `stopOnFirstError`
- `returnAllErrors`
- `customMessages`
- `removeAdditional`
- `useDefaults`
- `coerceTypes`
- `strict`
- `strictNumbers`
- `strictTypes`

The primary values used for validation are `data` and `schema`.

### Successful Output

When validation succeeds, the node returns the validated data.

For example:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

The returned data can contain transformations produced by the enabled validation options.

For example:

- string values can be converted to numbers when `coerceTypes` is enabled;
- missing default values can be added when `useDefaults` is enabled;
- additional properties can be removed when `removeAdditional` is enabled.

### Validation Error

When validation fails, the node throws a validation error.

For example:

```text
Validation failed: The name must be at least 2 characters.
```

When multiple errors are returned, the error can contain several validation messages:

```text
Validation failed:
The name must be at least 2 characters.;
The email must be a valid email.;
The age must be at least 18.
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: JSON Schema Validation

**Configuration**

```text
validationMode: jsonSchema
stopOnFirstError: false
returnAllErrors: true
removeAdditional: false
useDefaults: true
coerceTypes: true
strict: false
strictNumbers: false
strictTypes: false
```

**Data**

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

**Schema**

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "minLength": 2
    },
    "age": {
      "type": "integer",
      "minimum": 18
    },
    "email": {
      "type": "string",
      "format": "email"
    }
  },
  "required": [
    "name",
    "age",
    "email"
  ],
  "additionalProperties": false
}
```

**Output**

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

### Example 2: Multiple Validation Errors

**Data**

```json
{
  "name": "H",
  "age": 15,
  "email": "not-an-email"
}
```

With:

```text
stopOnFirstError: false
returnAllErrors: true
```

validation fails with errors for:

- `name`
- `age`
- `email`

Example error:

```text
Validation failed:
The name must be at least 2 characters.;
The age must be at least 18.;
The email must be a valid email.
```

### Example 3: Stop On First Error

Using the same invalid data:

```json
{
  "name": "H",
  "age": 15,
  "email": "not-an-email"
}
```

with:

```text
stopOnFirstError: true
```

the node stops at the first validation failure.

Example:

```text
The name must be at least 2 characters.
```

### Example 4: Type Coercion

**Data**

```json
{
  "name": "Hamza",
  "age": "25",
  "email": "hamza@example.com"
}
```

With:

```text
coerceTypes: true
```

and `age` defined as an integer in the JSON Schema, the output is:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

### Example 5: Remove Additional Properties

**Data**

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com",
  "secret": "should-be-removed"
}
```

With:

```text
removeAdditional: true
```

and:

```json
{
  "additionalProperties": false
}
```

the output is:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

### Example 6: Apply Default Values

**Data**

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com"
}
```

With a schema containing:

```json
{
  "country": {
    "type": "string",
    "default": "Morocco"
  }
}
```

and:

```text
useDefaults: true
```

the output contains the default value:

```json
{
  "name": "Hamza",
  "age": 25,
  "email": "hamza@example.com",
  "country": "Morocco"
}
```

### Example 7: Laravel-Style Validation

**Configuration**

```text
validationMode: laravel
stopOnFirstError: false
returnAllErrors: true
```

**Data**

```json
{
  "name": "Hamza",
  "email": "hamza@example.com",
  "age": 25
}
```

**Schema**

```json
{
  "name": "required|string|min:2",
  "email": "required|email",
  "age": "required|integer|min:18"
}
```

**Output**

```json
{
  "name": "Hamza",
  "email": "hamza@example.com",
  "age": 25
}
```

### Example 8: Laravel Validation Failure

**Data**

```json
{
  "name": "H",
  "email": "invalid-email",
  "age": 15
}
```

Using:

```json
{
  "name": "required|string|min:2",
  "email": "required|email",
  "age": "required|integer|min:18"
}
```

validation fails with messages for the three invalid fields.

Example:

```text
Validation failed:
The name must be at least 2 characters.;
The email must be a valid email.;
The age must be at least 18.
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Schema Validate Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Validation Failed

**Cause:** One or more values do not satisfy the configured validation rules.

Example:

```text
Validation failed: The name must be at least 2 characters.
```

**Solution:** Check the reported field and update the input data or validation schema as appropriate.

### Multiple Validation Errors

When:

```text
stopOnFirstError: false
returnAllErrors: true
```

the node can report multiple validation errors in a single execution.

This is expected behavior.

If only the first error should be returned, enable:

```text
stopOnFirstError: true
```

### Unexpected Type Conversion

When:

```text
coerceTypes: true
```

compatible input values can be converted to the types required by the JSON Schema.

For example:

```json
{
  "age": "25"
}
```

can become:

```json
{
  "age": 25
}
```

If automatic conversion is not desired, disable `coerceTypes`.

### Additional Properties Disappear

When `removeAdditional` is enabled and the JSON Schema does not permit additional properties, undeclared fields can be removed from the returned data.

For example:

```json
{
  "secret": "should-be-removed"
}
```

can be removed during validation.

Disable `removeAdditional` if these properties must be preserved.

### Missing Property Appears in Output

When `useDefaults` is enabled, properties with JSON Schema default values can be inserted automatically.

For example:

```json
{
  "country": {
    "type": "string",
    "default": "Morocco"
  }
}
```

can add:

```text
country: Morocco
```

to the returned data.

Disable `useDefaults` if defaults should not be inserted automatically.

### Laravel Rules Do Not Behave Like JSON Schema

The two validation modes use different schema formats.

For JSON Schema, use definitions such as:

```json
{
  "type": "string",
  "minLength": 2
}
```

For Laravel mode, use rules such as:

```text
required|string|min:2
```

Make sure `validationMode` matches the format used in `schema`.

### Email Validation Failure

A rule such as:

```text
email
```

or a JSON Schema format such as:

```json
{
  "format": "email"
}
```

requires a valid email value.

For example:

```text
invalid-email
```

fails validation.

### Strict Validation Issues

The node exposes:

```text
strict
strictNumbers
strictTypes
```

If validation becomes more restrictive than expected, check whether one or more strict options are enabled.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Validate Phone** — Validate phone number data.
- **Validate URL** — Validate URL values.
- **HTML Sanitize** — Sanitize HTML content before downstream processing.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for the Schema Validate node. |

<!-- /SECTION: changelog -->
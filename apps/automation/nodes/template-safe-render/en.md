---
node_id: "template-safe-render"
title: "Template Safe Render"
description: "Safely render templates using variable substitution without evaluating code."
category: "Utility / Data Transformation"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:

- template
- rendering
- variables
- substitution
- safe-render

related_nodes:
- function
- if
- stringify-json
---

# Template Safe Render

> **Category:** Utility / Data Transformation | **Type:** Action Node

Safely render a text template by replacing `{{variable}}` placeholders with values from configured variables or workflow input data.

The **Template Safe Render** node performs variable substitution only. It does not evaluate JavaScript expressions or use `eval`.

## Supported Features

- Render templates using `{{variable}}` syntax
- Access nested values using `{{nested.path}}`
- Use configured variables
- Use workflow input data directly as template variables
- Access the complete input through the special `item` variable
- Parse JSON strings automatically when possible
- Support arrays and objects as replacement values
- Support JavaScript `Map` objects when resolving values
- Safely replace missing or null variables with an empty string
- Handle object keys containing literal dots

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `data` | `string` | ❌ No | — | Optional input data used when workflow input is not an array and this value is provided. |
| `template` | `string` | ✅ Yes | — | Template string containing `{{variable}}` or `{{nested.path}}` placeholders. |
| `variables` | `record<string, unknown>` | ❌ No | — | Additional variables that can be referenced by the template. |

---

## Template Syntax

The node recognizes placeholders using the following pattern:

```text
{{variable}}
```

Whitespace around the variable name is supported:

```text
{{ variable }}
```

Nested properties can be accessed using dot notation:

```text
{{user.name}}
```

The expression inside `{{...}}` is trimmed before it is resolved.

### Example

Template:

```text
Hello {{name}}!
```

Variables:

```json
{
  "name": "John"
}
```

Output:

```text
Hello John!
```

---

## Input Data

The node determines which input data to use according to the following order:

1. If the workflow input is an array, the array is used.
2. Otherwise, if the configured `data` value is defined and not empty, it is used.
3. Otherwise, the workflow input is used.

### JSON String Input

When the selected input is a string, the node attempts to parse it as JSON.

For example:

```text
{"name":"John","age":30}
```

is parsed into an object and can then be referenced using:

```text
{{name}}
{{age}}
```

If JSON parsing fails, the value is treated as a normal string.

---

## Variable Resolution

The node creates a variable collection from the configured `variables` and the selected input data.

Configured variables are initially copied into the variable collection.

### Object Input

When the input is an object, its properties are added directly to the available variables.

For example:

```json
{
  "name": "John",
  "age": 30
}
```

allows:

```text
{{name}}
{{age}}
```

The complete object is also available under the `item` variable:

```text
{{item.name}}
{{item.age}}
```

### Non-Object Input

For values that are not objects, the value is available through `item`.

For example, if the input is:

```text
Hello World
```

the template can use:

```text
{{item}}
```

---

## Nested Properties

Nested properties can be accessed using dot notation.

Given:

```json
{
  "user": {
    "name": "John",
    "email": "john@example.com"
  }
}
```

the following template:

```text
Name: {{user.name}}
Email: {{user.email}}
```

produces the corresponding values.

The same nested access is available through `item`:

```text
Name: {{item.user.name}}
```

---

## Literal Dot Keys

The resolver first checks whether the complete key exists directly before interpreting dots as nested paths.

This means an object such as:

```json
{
  "user.name": "John"
}
```

can be accessed with:

```text
{{user.name}}
```

The direct key is resolved before the node attempts nested-property traversal.

---

## Map Support

The value resolver supports JavaScript `Map` objects.

If the current object is a `Map` and contains the requested key, its value is retrieved using `Map.get()`.

This behavior applies both when resolving the initial key and when traversing nested properties.

---

## Object and Array Values

Primitive values are converted to strings using `String()`.

Objects and arrays are converted using `JSON.stringify()`.

For example:

```json
{
  "user": {
    "name": "John",
    "age": 30
  }
}
```

with:

```text
User: {{user}}
```

returns the JSON representation of the `user` object.

Arrays are handled in the same way:

```text
Items: {{items}}
```

returns the JSON representation of the array.

---

## Missing Variables

If a variable cannot be resolved, the placeholder is replaced with an empty string.

For example:

```text
Hello {{name}}!
```

when `name` is not available becomes:

```text
Hello !
```

The node does not throw an error for missing variables.

---

## Null Values

Variables resolving to `null` are also replaced with an empty string.

For example:

```text
Value: {{value}}
```

with:

```json
{
  "value": null
}
```

produces:

```text
Value:
```

---

## Inputs & Outputs

### Inputs

The node accepts workflow input but does not require workflow input.

Configuration can provide:

- `template`
- `variables`
- optional `data`

When no configured `data` is provided, the workflow input is used.

### Outputs

The node returns the rendered template as a string.

Example:

**Template:**

```text
Hello {{name}}, welcome to {{company}}.
```

**Variables:**

```json
{
  "name": "John",
  "company": "Fusion"
}
```

**Output:**

```text
Hello John, welcome to Fusion.
```

---

## Rendering Behavior

The node scans the template for all expressions matching:

```text
{{...}}
```

Each expression is resolved independently.

For every expression:

1. Whitespace is removed from the beginning and end.
2. The expression is resolved against the available variables.
3. Missing or null values become an empty string.
4. Objects and arrays are serialized as JSON.
5. Primitive values are converted to strings.
6. The resolved value replaces the original placeholder.

The rendered template is returned as a string.

---

## Security

The node is designed for safe template rendering.

It performs **variable substitution only** and does not evaluate expressions as JavaScript.

For example:

```text
{{name}}
```

is treated as a variable lookup.

The node does not provide template execution through `eval` or similar JavaScript evaluation.

---

## Error Handling

### Missing Template

```text
Template is required
```

Raised when the `template` value is empty or unavailable.

The `template` parameter is required by the schema.

### Invalid JSON Input

Invalid JSON does not produce an error.

If the selected input is a string and `JSON.parse()` fails, the original string is retained and processed as plain input data.

---

## Configuration Examples

### Basic Rendering

```json
{
  "template": "Hello {{name}}!",
  "variables": {
    "name": "John"
  }
}
```

Result:

```text
Hello John!
```

### Nested Variables

```json
{
  "template": "User: {{user.name}}",
  "variables": {
    "user": {
      "name": "John"
    }
  }
}
```

Result:

```text
User: John
```

### Using Workflow Input

Input:

```json
{
  "name": "John",
  "role": "Developer"
}
```

Template:

```text
{{name}} is a {{role}}.
```

Result:

```text
John is a Developer.
```

### Using `item`

Input:

```json
{
  "user": {
    "name": "John"
  }
}
```

Template:

```text
User: {{item.user.name}}
```

Result:

```text
User: John
```

### Object Replacement

Variables:

```json
{
  "user": {
    "name": "John",
    "age": 30
  }
}
```

Template:

```text
User data: {{user}}
```

The object is inserted as its JSON representation.

---

## Workflow Integration

### Function → Template Safe Render

A common pattern is to generate structured data with a Function node and then use Template Safe Render to produce a final message.

```text
Function → Template Safe Render
```

The Function node can provide values such as:

```json
{
  "name": "John",
  "status": "completed"
}
```

The Template Safe Render node can then use:

```text
Hello {{name}}, your request is {{status}}.
```

### HTTP/Data Source → Template Safe Render

Structured data received from another workflow node can be used directly:

```text
Data Source → Template Safe Render
```

The input fields become available as template variables.

### Template Safe Render → If

The rendered string can be passed to an `If` node for subsequent workflow logic:

```text
Template Safe Render → If
```

---

## Common Patterns

- Function → Template Safe Render — convert generated data into readable text
- HTTP Request → Template Safe Render — format API response data
- Data processing → Template Safe Render — create messages or documents from structured values
- Template Safe Render → If — render content before applying workflow conditions

---

## Troubleshooting

### Placeholder Is Replaced With an Empty String

**Cause**

The requested variable does not exist, or its value is `null` or `undefined`.

**Solution**

Check that the variable exists in either:

- `variables`
- workflow input
- `item`

For nested values, verify the property path:

```text
{{user.name}}
```

### Nested Property Does Not Resolve

**Cause**

One of the properties in the path does not exist or an intermediate value is not an object.

**Solution**

Verify the input structure and the complete path.

For example:

```text
{{item.user.name}}
```

requires `item.user` to contain a `name` property.

### JSON String Is Not Parsed

**Cause**

The selected string is not valid JSON.

**Solution**

The node automatically falls back to treating the value as plain text. If structured access is required, provide valid JSON.

### Template Is Not Rendering

**Cause**

The placeholder does not match the supported `{{...}}` syntax or the variable cannot be found.

**Solution**

Use supported syntax such as:

```text
{{name}}
```

or:

```text
{{user.name}}
```

---

## Notes

The node:

- Supports variable substitution only.
- Does not evaluate JavaScript expressions.
- Does not execute template code.
- Supports simple and nested variable paths.
- Makes object input available both directly and through `item`.
- Makes non-object input available through `item`.
- Attempts to parse string input as JSON.
- Serializes objects and arrays with `JSON.stringify()`.
- Replaces missing and null values with an empty string.
- Supports JavaScript `Map` objects during key resolution.
- Checks direct keys before interpreting dots as nested paths.
- Returns the final rendered template as a string.

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-27 | Initial release |

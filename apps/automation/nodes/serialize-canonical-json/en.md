---
node_id: "serialize-canonical-json"
title: "Serialize Canonical JSON"
description: "Serialize JSON with stable key ordering."
category: "data-transformation-etl"
subcategory: "serialization-conversion"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - json
  - serialize
  - canonical
  - hashing
  - data-transformation
  - key-sorting
related_nodes:
  - parse-json
  - checksum-calculator
  - hash
  - hmac
  - default-fill
---

<!-- SECTION: header -->
# Serialize Canonical JSON

> **Category:** Data Transformation (ETL) | **Subcategory:** Serialization & Conversion | **Type:** Action Node

Serialize JSON data into a compact, deterministic string with stable lexicographical key ordering.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Serialize Canonical JSON** node converts JSON objects, arrays, and primitive data structures into a standardized ("canonical") JSON string format.

In standard JSON serialization, object key ordering is non-deterministic and can vary across different runtimes or execution steps. This node recursively sorts all object keys in alphabetical order (`A-Z`) before stringifying, ensuring that identical data structures always produce the exact same byte string.

### Key Features

- **Deterministic Key Sorting:** Recursively sorts all object keys alphabetically across nested structures.
- **Compact Output:** Returns a space-free, compact JSON string suitable for cryptographic operations.
- **Flexible Input Sources:** Accepts configuration strings, incoming payload objects, arrays, or primitive values.
- **Smart String Parsing:** Automatically parses standard JSON strings and JavaScript object literal formats (e.g. `{b: 2, a: 1}`).
- **Safe Expression Fallback:** Utilizes a secure expression evaluator to safely interpret unquoted JS object strings without security risks.
- **Array Preservation:** Retains original array index order while recursively canonicalizing object items inside arrays.

### Processing Flow

```text
Input Payload / Config Data
            ↓
Resolve Data Source (Array > Config String > Input Port Payload)
            ↓
    Is Data a String?
   ├── Yes ──→ Trim Whitespace
   │             ↓
   │           JSON.parse() succeeds?
   │             ├── Yes ─→ Parsed JSON Object
   │             └── No  ─→ Secure Expression Evaluator ({a: 1}) succeeds?
   │                          ├── Yes ─→ Evaluated Object
   │                          └── No  ─→ Treat as Plain String ─→ JSON.stringify(trimmed)
   └── No  ──→ Use Raw Input Data (Object, Array, Primitive, undefined)
            ↓
    If Undefined? ──→ Return "null"
            ↓
Recursive Canonicalization (canonicalize)
   ├── Primitive / Null ──→ Return as-is
   ├── Array ────────────→ Map canonicalize() over elements
   └── Object ───────────→ Sort Object.keys() alphabetically & map values
            ↓
Return Compact JSON String: JSON.stringify(canonical)
```

### Use Cases

- **Cryptographic Hashing & Signatures:** Prepare payload objects for HMAC, MD5, or SHA256 signing where key order consistency is strictly required.
- **Cache Key Generation:** Generate identical hash keys for API caching regardless of how upstream nodes order object fields.
- **Data Integrity Verification:** Compare payload snapshots across microservices to detect true semantic changes versus key reordering.
- **Normalized Data Storage:** Ensure database JSON strings are formatted uniformly for string matching or auditing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `string` | ❌ No | — | Optional data string to serialize (JSON string or object literal). If omitted, incoming workflow payload from the input port is used. |

---

### Parameter Details

#### `data`
Optional input string supplied in the node configuration panel.
- **Type:** `string`
- **Required:** No
- **Supported Formats:**
  - Standard JSON strings: `{"z": 10, "a": 5, "m": 1}`
  - JS Object Literal strings: `{b: 2, a: 1}`
  - JSON Arrays: `[{"c": 3, "b": 2}, {"y": 2, "x": 1}]`
  - Plain strings or numbers
- **Dynamic Bindings:** Can reference upstream node outputs using expressions like `{{outputs.HTTP_Request.success.body}}`.

---

### Input Priority Resolution

The node resolves the source data using the following strict evaluation hierarchy:

1. **Array Data:** If incoming payload from the input port (`data`) is an array, it is used immediately.
2. **Configured Parameter:** If the `data` parameter in configuration is populated (not `undefined`, `null`, or `""`), it is parsed and processed.
3. **Input Port Payload:** Otherwise, any payload received from the upstream node input port is processed.
4. **Undefined Fallback:** If no input data is provided (`undefined`), the node returns the canonical string `"null"`.

---

### Key Sorting & Canonicalization Rules

1. **Object Keys:** All keys in an object are extracted and sorted using standard alphabetical sort (`Object.keys(obj).sort()`).
2. **Nested Objects:** Object key sorting is applied recursively down to all nested child objects.
3. **Arrays:** Array elements preserve their original index position. Objects contained inside arrays undergo key sorting.
4. **Formatting:** The output string contains no unnecessary whitespace, line breaks, or indentation.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload (object, array, string, or primitive). |

---

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` | Returns the canonical compact JSON string with alphabetically sorted keys. |
| `error` | `Error` | Emitted if an unexpected runtime evaluation error occurs. |

---

### Canonicalization Examples

#### Input Object with Unsorted Keys

```json
{
  "z": 10,
  "a": 5,
  "m": 1
}
```

#### Serialized Output String

```json
"{\"a\":5,\"m\":1,\"z\":10}"
```

---

#### Nested Object Input

```json
{
  "user": {
    "name": "Ali",
    "age": 25
  },
  "active": true
}
```

#### Serialized Output String

```json
"{\"active\":true,\"user\":{\"age\":25,\"name\":\"Ali\"}}"
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Basic Unsorted JSON String

Standard JSON string with arbitrary key ordering configured in the `data` parameter.

**Configuration:**

```text
Data: {"z": 10, "a": 5, "m": 1}
```

**Output:**

```json
"{\"a\":5,\"m\":1,\"z\":10}"
```

---

### Example 2: Nested Object Structure

Canonicalization of deep nested user data structures.

**Configuration:**

```text
Data: {"user": {"name": "Ali", "age": 25}, "active": true}
```

**Output:**

```json
"{\"active\":true,\"user\":{\"age\":25,\"name\":\"Ali\"}}"
```

---

### Example 3: Array of Objects

Sorting object keys within array items while preserving original array order.

**Configuration:**

```text
Data: [{"c": 3, "b": 2}, {"y": 2, "x": 1}]
```

**Output:**

```json
"[{\"b\":2,\"c\":3},{\"x\":1,\"y\":2}]"
```

---

### Example 4: Dynamic Upstream Workflow Input

Receiving payload dynamically from a preceding **Function** node or **HTTP Request** node.

**Upstream Node Output:**

```json
{
  "city": "Casablanca",
  "country": "Morocco"
}
```

**Serialize Canonical JSON Output:**

```json
"{\"city\":\"Casablanca\",\"country\":\"Morocco\"}"
```

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Serialize Canonical JSON Example Workflow
```

### How it flows

1. **Manual Trigger:** Triggers the workflow execution branches.
2. **Serialize Canonical JSON:** Evaluates object inputs (single objects, nested structures, arrays, dynamic upstream data).
3. **Log Node:** Prints the serialized canonical JSON strings to the execution console.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Pairing with Hashing Nodes:** Use this node immediately prior to a **Checksum Calculator**, **Hash**, or **HMAC** node to guarantee consistent signature outputs regardless of how data was constructed.
2. **Handling String Inputs:** If your data arrives as a stringified JS object literal (e.g. `{a: 1}` without double quotes around keys), the node automatically parses it securely without requiring manual string cleanup.
3. **Preserving Array Sequence:** Note that array element order is preserved (`[2, 1]` remains `[2, 1]`). Only object keys inside array items are sorted.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Questions

#### Why did my array order not change?
- **Explanation:** Canonical JSON specification only standardizes key ordering for **objects**. Array order represents ordered lists and is explicitly preserved.

#### What happens if input is undefined?
- **Explanation:** If no data is configured and no payload is received from the upstream node, the node safely returns the string `"null"`.

#### How are plain strings handled?
- **Explanation:** If an input string cannot be parsed as JSON or evaluated as a JS object, it is treated as a plain text string and wrapped in JSON string quotes (e.g. `"\"hello\""`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Parse JSON](../parse-json/en.md) — Parse JSON strings into structured JavaScript objects
- [Checksum Calculator](../checksum-calculator/en.md) — Generate MD5, SHA256, and SHA512 hashes
- [Default Fill](../default-fill/en.md) — Fill missing object fields with default fallback values
- [Log](../log/en.md) — Output workflow data to execution logs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial release of the Serialize Canonical JSON node with stable key ordering and secure string parsing |

<!-- /SECTION: changelog -->

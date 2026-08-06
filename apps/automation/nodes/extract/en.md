---
node_id: "extract"
title: "Extract"
description: "Extract substring or regex groups from string"
category: "Data Transformation (ETL)"
subcategory: "Data Shaping"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - text
  - string
  - extract
  - regex
related_nodes:
  - function
  - replace
  - regex-match
---

<!-- SECTION: header -->
# Extract

> **Category:** Utilities | **Type:** Action Node

Extract a substring or regex capture group from a string value.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Extract** node helps retrieve a portion of a string or a regex-matched group from incoming text. It is useful for parsing identifiers, codes, values, or tokens from larger messages.

### Key Features

- **Substring Extraction:** Extract a portion of text using start and end positions
- **Regex Support:** Retrieve one or more regex capture groups from input text
- **Simple Configuration:** Use a small set of parameters for fast parsing
- **Workflow Friendly:** Combine it with downstream formatting or validation nodes

### Use Cases

- Extract confirmation codes from messages
- Get a specific segment from a longer string
- Parse values from logs or API responses
- Capture text matching a regex pattern

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | `enum` | ✅ Yes | `substring` | Extraction mode: `substring` or `regex` |
| `data` | `string` | ✅ Yes | — | Source string to analyze |
| `start` | `number` | ❌ No | `0` | Start position used in substring mode |
| `end` | `number` | ❌ No | `0` | End position used in substring mode |

### Substring Mode

In `substring` mode, the node extracts text from the input string using the configured start and end parameters.

Example:

```text
data: "Le code de confirmation est 8492."
start: 30
end: 2
```

This extracts the targeted portion of the original string according to the configured boundaries.

### Regex Mode

In `regex` mode, the node extracts one or more values from the input text based on a regular expression pattern. This is useful when the target value is embedded inside a larger message and follows a predictable structure.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data that can be used as the source string |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` | The extracted text |
| `error` | `object` | Error details if extraction fails |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Extract a Confirmation Code

Use substring mode to pull a code from a sentence.

**Configuration:**

```text
mode: substring
data: "Le code de confirmation est 8492."
start: 30
end: 2
```

**Result:**

```text
8492
```

### Example: Regex Capture

Use regex mode when the string contains structured content such as an ID or label.

```text
Input: "Order #12345 was confirmed"
Pattern: "#(\\d+)"
```

**Result:**

```text
12345
```

<!-- /SECTION: examples -->

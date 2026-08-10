---
node_id: "astm-to-json"
title: "ASTM to JSON"
description: "Convert ASTM E1394 / E1381 formatted laboratory messages into a structured JSON representation."
category: "Data Transformation (ETL)"
subcategory: "EDI & Structured Messages"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - astm
  - edi
  - lab
  - transform
related_nodes:
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# ASTM to JSON

> **Category:** Data Transformation (ETL) | **Subcategory:** EDI & Structured Messages | **Type:** Action Node

Convert laboratory instrument messages in ASTM E1394/E1381 format into a normalized JSON structure that can be used by downstream automation flows or stored in databases.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **ASTM to JSON** node parses legacy ASTM-formatted lab messages (commonly used by Analyzer devices and Laboratory Information Systems) and produces a structured JSON object containing header, patient, order, and result sections.

It supports common AST​​M record types such as `H` (Header), `P` (Patient), `O` (Order), `R` (Result), and `L` (Termination). The node is resilient to minor formatting variations and exposes parsing errors through the `error` output.

### Key Features

- Parse `H`, `P`, `O`, `R`, and `L` segments into JSON
- Normalize patient identifiers, names, and demographic fields
- Extract ordered tests, results, units, reference ranges, flags, and timestamps
- Provide a compact JSON payload optimized for downstream ingestion
- Error detection and descriptive error messages for malformed segments

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `astmData` | `string` | ✅ Yes | — | Raw ASTM message text. Records may be separated by line breaks or the `\r` character. |
| `preserveRaw` | `boolean` | No | `false` | If true, include the original raw `astmData` string in the output JSON under `raw`. |
| `timeFormat` | `string` | No | `YYYYMMDDhhmmss` | Expected timestamp format in header/order/result records. |

### Input Format

Provide the complete ASTM message as a single string in `astmData`. Example record separators include carriage return `\r` or newline `\n`. Example message:

```
H|\^&|||Analyzer_01|||||||P|1|20260810111500
P|1||PAT12345||Smith^John||19850515|M
O|1|SPEC67890||^^^GLU^Glucose\^^^CHO^Cholesterol
R|1|^^^GLU^Glucose|105|mg/dL|70-99|H||F|||20260810101500
R|2|^^^CHO^Cholesterol|185|mg/dL|<200|N||F|||20260810101500
L|1|N
```

The node will parse each segment and map fields into a JSON structure described below.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data; node reads the `astmData` parameter if present, otherwise reads `input.astmData`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Parsed JSON object with `header`, `patient`, `orders`, and `results` sections |
| `error` | `Error` | Emitted when parsing fails or required fields are missing |

### Output JSON Structure

```json
{
  "header": {
    "sending_application": "Analyzer_01",
    "message_datetime": "2026-08-10T11:15:00Z"
  },
  "patient": {
    "id": "PAT12345",
    "last_name": "Smith",
    "first_name": "John",
    "dob": "1985-05-15",
    "gender": "M"
  },
  "orders": [
    {
      "id": "SPEC67890",
      "tests": [
        {"code": "GLU", "name": "Glucose"},
        {"code": "CHO", "name": "Cholesterol"}
      ]
    }
  ],
  "results": [
    {"test_code": "GLU", "value": "105", "units": "mg/dL", "reference_range": "70-99", "flag": "H", "timestamp": "2026-08-10T10:15:00Z"},
    {"test_code": "CHO", "value": "185", "units": "mg/dL", "reference_range": "<200", "flag": "N", "timestamp": "2026-08-10T10:15:00Z"}
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Parse ASTM Message

**Configuration:** set `astmData` to the sample message shown in the Configuration section.

**Output:** The `success` output emits the JSON structure shown above.

### Handling Malformed Messages

If a required segment is missing (for example, no `P` patient segment), the node emits an `error` with a descriptive message indicating which segment or field failed to parse.

---

## Notes

- The node does not validate clinical content beyond field presence and basic formatting.
- When integrating with downstream systems, ensure timestamps and identifiers are normalized to your platform's conventions.
- Use `preserveRaw` to keep the original message for audit or debugging purposes.

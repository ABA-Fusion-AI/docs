---
node_id: "edi-parse-hl7"
title: "Advanced HL7 v2.x Parser"
description: "Advanced HL7 v2.x parser with intelligent JSON conversion, field mapping, repetition handling, and multiple output formats (structured, flat, nested)."
category: "data-transformation-etl"
subcategory: "edi-structured-messages"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:
  - edi
  - hl7
  - healthcare
  - v2
  - parser
  - json
  - structured-data
  - integration
  - action
related_nodes:
  - edi-generate-hl7
  - edi-generate-edifact
  - edi-parse-edifact
  - edi-generate-x12
  - edi-parse-x12
---

<!-- SECTION: overview -->
# Advanced HL7 v2.x Parser

> **Category:** Data Transformation & ETL  | **Type:** Action Node

Parse, validate, and convert Health Level 7 (HL7 v2.x) raw healthcare messages (e.g., ADT, ORU, ORM, MDM) into structured JSON formats.

The **Advanced HL7 v2.x Parser** node processes HL7 v2.x pipe-delimited text, auto-detects encoding characters from the `MSH` header, unescapes special sequence characters (`\F\`, `\S\`, `\T\`, `\R\`, `\E\`), parses field components, subcomponents, and repetitions (`~`), and transforms the message into flexible JSON structures (`structured`, `flat`, or `nested`).

### Use Cases

- Parse incoming HL7 ADT (Admission, Discharge, Transfer) messages into JSON for EHR/EMR integration
- Extract patient demographics (`PID`), visit info (`PV1`), and clinical data (`OBR`, `OBX`)
- Transform healthcare EDI payloads for API endpoints or cloud databases
- Validate HL7 message header completeness (`MSH-9`, `MSH-10`, `MSH-12`) before processing downstream
- Convert HL7 raw messages into flat key-value pairs for easy expression referencing in workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputData` | `string` | ✅ Yes | `""` | Raw HL7 v2.x message string to parse |
| `validate` | `boolean` | ❌ No | `true` | Validate mandatory HL7 header fields and segment ordering |
| `outputFormat` | `enum` | ❌ No | `"structured"` | Output format structure (`"structured"`, `"flat"`, `"nested"`) |
| `encoding` | `string` | ❌ No | `"UTF-8"` | Character encoding of input data |
| `strictMode` | `boolean` | ❌ No | `false` | Throw an exception if validation errors occur |
| `includeEmptyFields` | `boolean` | ❌ No | `false` | Preserve nulls/empty entries for missing fields or components |
| `prettyPrint` | `boolean` | ❌ No | `true` | Format the output JSON cleanly |

### Output Formats

| Format | Description |
|--------|-------------|
| `structured` | Full hierarchical object containing parsed metadata header, grouped segments with detailed sequence/fields/components, raw message, and validation report. |
| `flat` | Simplified flat key-value dictionary using dot-notation (e.g., `PID.3.1`, `PV1[0].3`) for quick field extraction in expressions. |
| `nested` | Clean hierarchical structure separating metadata into a `header` object and grouping message payload into a `segments` object. |

---

### Output Format Comparison

Given the sample HL7 message segment:
```text
MSH|^~\&|HIS|HOSPITAL|LAB|HOSPITAL|20260806200000||ADT^A01|123456|P|2.5
PID|1||123456^^^HOSPITAL||DOE^JOHN||19900101|M
```

#### 1. Structured Format (`"structured"`)

```json
{
  "messageType": {
    "type": "ADT",
    "event": "A01",
    "full": "ADT^A01"
  },
  "messageControlId": "123456",
  "version": "2.5",
  "processingId": "P",
  "timestamp": "20260806200000",
  "sendingApplication": "HIS",
  "sendingFacility": "HOSPITAL",
  "receivingApplication": "LAB",
  "receivingFacility": "HOSPITAL",
  "segments": {
    "MSH": [
      {
        "segment": "MSH",
        "sequence": 0,
        "raw": "MSH|^~\\&|HIS|HOSPITAL|LAB|HOSPITAL|20260806200000||ADT^A01|123456|P|2.5",
        "fields": [
          { "value": "|" },
          { "value": "^~\\&" },
          { "value": "HIS" },
          { "value": "HOSPITAL" }
        ]
      }
    ],
    "PID": [
      {
        "segment": "PID",
        "sequence": 1,
        "raw": "PID|1||123456^^^HOSPITAL||DOE^JOHN||19900101|M",
        "fields": [
          { "value": "1" },
          { "value": "" },
          {
            "value": "123456^^^HOSPITAL",
            "components": [
              { "value": "123456" },
              { "value": "" },
              { "value": "" },
              { "value": "HOSPITAL" }
            ]
          },
          { "value": "" },
          {
            "value": "DOE^JOHN",
            "components": [
              { "value": "DOE" },
              { "value": "JOHN" }
            ]
          }
        ]
      }
    ]
  },
  "raw": "MSH|^~\\&|...",
  "validation": {
    "isValid": true,
    "errors": [],
    "warnings": []
  }
}
```

#### 2. Flat Format (`"flat"`)

```json
{
  "messageType": "ADT^A01",
  "messageControlId": "123456",
  "version": "2.5",
  "timestamp": "20260806200000",
  "PID.1": "1",
  "PID.3.1": "123456",
  "PID.3.4": "HOSPITAL",
  "PID.5.1": "DOE",
  "PID.5.2": "JOHN",
  "PID.7": "19900101",
  "PID.8": "M"
}
```

#### 3. Nested Format (`"nested"`)

```json
{
  "header": {
    "messageType": {
      "type": "ADT",
      "event": "A01",
      "full": "ADT^A01"
    },
    "messageControlId": "123456",
    "version": "2.5",
    "processingId": "P",
    "timestamp": "20260806200000",
    "sending": {
      "application": "HIS",
      "facility": "HOSPITAL"
    },
    "receiving": {
      "application": "LAB",
      "facility": "HOSPITAL"
    }
  },
  "segments": {
    "PID": {
      "field_1": "1",
      "field_2": "",
      "field_3": ["123456", "", "", "HOSPITAL"],
      "field_5": ["DOE", "JOHN"],
      "field_7": "19900101",
      "field_8": "M"
    }
  }
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. The parser automatically extracts text from `data.data`, `data.message`, `data.hl7`, `data.content`, or `data.text` if `inputData` parameter is not set explicitly |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when parsing succeeds. Formatted according to `outputFormat` |
| `error` | `Error` | Emitted if message normalization, encoding extraction, or strict validation fails |

---

### Example: Parse HL7 to Flat Format

**Configuration**

```json
{
  "inputData": "MSH|^~\\&|HIS|HOSPITAL|LAB|HOSPITAL|20260806200000||ADT^A01|123456|P|2.5\rEVN|A01|20260806200000\rPID|1||123456^^^HOSPITAL||DOE^JOHN||19900101|M\rPV1|1|I|WARD^101^1||||1234^SMITH^JANE",
  "outputFormat": "flat",
  "validate": true,
  "strictMode": false
}
```

**Output**

```json
{
  "messageType": "ADT^A01",
  "messageControlId": "123456",
  "version": "2.5",
  "timestamp": "20260806200000",
  "EVN.1": "A01",
  "EVN.2": "20260806200000",
  "PID.1": "1",
  "PID.3.1": "123456",
  "PID.3.4": "HOSPITAL",
  "PID.5.1": "DOE",
  "PID.5.2": "JOHN",
  "PID.7": "19900101",
  "PID.8": "M",
  "PV1.1": "1",
  "PV1.2": "I",
  "PV1.3.1": "WARD",
  "PV1.3.2": "101",
  "PV1.3.3": "1",
  "PV1.7.1": "1234",
  "PV1.7.2": "SMITH",
  "PV1.7.3": "JANE"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Receive HL7 Message and Parse to Flat JSON

```json
{
  "nodes": [
    {
      "id": "manual-trigger",
      "type": "manual-trigger"
    },
    {
      "id": "get-hl7-message",
      "type": "function",
      "config": {
        "code": "return 'MSH|^~\\\\&|HIS|HOSPITAL|LAB|HOSPITAL|20260806200000||ADT^A01|123456|P|2.5\\rEVN|A01|20260806200000\\rPID|1||123456^^^HOSPITAL||DOE^JOHN||19900101|M\\rPV1|1|I|WARD^101^1||||1234^SMITH^JANE';"
      }
    },
    {
      "id": "parse-hl7",
      "type": "edi-parse-hl7",
      "config": {
        "outputFormat": "flat",
        "inputData": "{{output.node.get-hl7-message.success}}"
      }
    },
    {
      "id": "log-output",
      "type": "log"
    }
  ]
}
```

### Common Patterns

- Healthcare Webhook Receiver → Advanced HL7 v2.x Parser → PostgreSQL Database
- MLLP / TCP Listener → Advanced HL7 v2.x Parser → Route ADT vs ORU messages
- HL7 File Ingestion → Advanced HL7 v2.x Parser (Flat Format) → EHR System API Call
- HL7 Parser → EDI Generate HL7 (Message Transformation)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### HL7 message data is required

**Cause**

No string was passed to `inputData`, and the incoming node data object did not contain any property named `data`, `message`, `hl7`, `content`, or `text`.

**Solution**

Set the `inputData` parameter or map it dynamically using expressions (e.g. `{{input.body.rawMessage}}`).

---

### Invalid HL7 message: Must start with MSH segment

**Cause**

The raw input message does not begin with `MSH` segment header after normalizing line endings.

**Solution**

Ensure leading whitespace or invalid characters are trimmed, and confirm the message starts with `MSH`.

---

### MSH segment too short to extract encoding characters

**Cause**

The `MSH` header has fewer than 8 characters, making it impossible to read `MSH-2` encoding characters (`|^~\&`).

**Solution**

Verify the message header format. A valid `MSH` header must include the field separator and encoding characters (e.g., `MSH|^~\&|...`).

---

### HL7 validation failed

**Cause**

The `validate` parameter is `true`, `strictMode` is `true`, and mandatory fields or segment rules were violated (e.g., missing `MSH-9.1` Message Type, missing `MSH-10` Control ID, or multiple `MSH` segments).

**Solution**

- Review error details in the `validation.errors` output array.
- Turn off `strictMode` if you want to allow non-strict messages to process with validation warnings instead of throwing an error.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [EDI Generate HL7](../edi-generate-hl7/en.md) — Generate HL7 v2.x healthcare messages from JSON
- [EDI Parse EDIFACT](../edi-parse-edifact/en.md) — Parse UN/EDIFACT documents into JSON
- [EDI Generate EDIFACT](../edi-generate-edifact/en.md) — Generate UN/EDIFACT documents from JSON
- [EDI Parse X12](../edi-parse-x12/en.md) — Parse ANSI ASC X12 documents into JSON
- [EDI Generate X12](../edi-generate-x12/en.md) — Generate ANSI ASC X12 documents from JSON

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-11 | Initial release |

<!-- /SECTION: changelog -->

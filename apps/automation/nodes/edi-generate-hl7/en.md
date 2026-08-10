---
node_id: "edi-generate-hl7"
title: "EDI Generate HL7"
description: "Generate HL7 v2.x healthcare messages from structured JSON data."
category: "data-transformation-etl"
subcategory: "edi-structured-messages"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - edi
  - hl7
  - healthcare
  - v2
  - structured-data
  - integration
related_nodes:
  - edi-parse-hl7
  - edi-generate-edifact
  - edi-parse-edifact
  - edi-generate-x12
  - edi-parse-x12
---

<!-- SECTION: header -->
# EDI Generate HL7

> **Category:** Data Transformation & ETL | **Type:** Action Node

Generate HL7 v2.x healthcare messages (e.g., ADT, ORU, ORM) from structured JSON objects. Converts message segments, fields, components, subcomponents, and repetition groups into standard HL7 formatted strings separated by carriage return characters (`\r`).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EDI Generate HL7** node generates standards-compliant Health Level 7 (HL7 v2.x) messages from structured JSON representations.

Internally, the node uses `HL7Parser.generate()` to construct the message. It processes the specified delimiters and segment array, encoding fields, components (`^`), repetitions (`~`), and subcomponents (`&`) into standard pipe-delimited (`|`) HL7 segments.

### Key Features

- **Structured JSON Input:** Build HL7 messages programmatically using clean JSON data.
- **Configurable Delimiters:** Customize field (`|`), component (`^`), repetition (`~`), escape (`\`), and subcomponent (`&`) characters.
- **Segment-Based Construction:** Add standard or custom segments (`MSH`, `EVN`, `PID`, `PV1`, `OBR`, `OBX`, etc.) sequentially.
- **Hierarchical Encoding:**
  - Simple scalar values for basic fields
  - Arrays for component-separated values (`^`)
  - Nested arrays for repetition (`~`) and subcomponents (`&`)
- **Flexible Data Source:** Provide data directly in node parameters or dynamically from preceding workflow nodes.
- **Standardized Output:** Returns an object containing the generated HL7 text string and format identifier (`"HL7"`).

### Message Structure

A generated HL7 v2.x message consists of segments separated by carriage returns (`\r`):

```text
MSH|^~\&|SendingApp|SendingFacility|ReceivingApp|ReceivingFacility|20260810153000||ADT^A01|MSG00001|P|2.5
EVN|A01|20260810153000
PID|1||PAT12345||DOE^JOHN^A||19800101|M
PV1|1|I|FLAG^200^1
```

### Standard Delimiters

| Delimiter | Purpose | Default Character |
|-----------|---------|-------------------|
| Field | Separates fields within a segment | `|` |
| Component | Separates components within a field | `^` |
| Repetition | Separates repeated values within a field | `~` |
| Escape | Escape character for special symbols | `\` |
| Subcomponent | Separates subcomponents within a component | `&` |

### Use Cases

- **EHR/EMR Integration:** Generate patient admission (`ADT^A01`), discharge (`ADT^A03`), or transfer (`ADT^A02`) messages.
- **Laboratory Information Systems (LIS):** Construct lab order (`ORM^O01`) and lab result (`ORU^R01`) messages.
- **Pharmacy & Radiology Interoperability:** Build scheduling (`SIU`), pharmacy (`RDE`), or imaging order payloads.
- **Healthcare Data Transformation:** Convert REST API JSON payloads or database records into HL7 v2.x format.
- **HIE & Interface Engines:** Prepare outbound HL7 payloads for Mirth Connect, Cloverleaf, or hospital network endpoints.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `any` | Conditional | — | Structured JSON object representing the HL7 message structure. If empty, the node reads from incoming workflow data. |

---

### JSON Data Structure

The `data` input object must contain two primary properties: `delimiters` and `segments`.

```typescript
interface HL7InputData {
  delimiters: {
    field: string;         // Field separator, usually "|"
    component: string;     // Component separator, usually "^"
    repetition: string;    // Repetition separator, usually "~"
    escape: string;        // Escape character, usually "\"
    subcomponent: string;  // Subcomponent separator, usually "&"
  };
  segments: Array<{
    id: string;            // Segment header tag (e.g. "MSH", "PID", "PV1")
    fields: Array<any>;    // Array of field values (strings, arrays for components/repetitions)
  }>;
}
```

---

### Field Formatting Rules

The generator formats fields based on their JavaScript data type inside `fields`:

| Data Structure | Output Format | Example Input | Generated HL7 |
|----------------|---------------|---------------|---------------|
| `null` / `undefined` / `""` | Empty field | `""` | `\|\|` |
| `string` / `number` | Simple field value | `"PAT12345"` | `\|PAT12345\|` |
| `Array<string>` | Component-separated field (`^`) | `["DOE", "JOHN", "A"]` | `\|DOE^JOHN^A\|` |
| `Array<Array<string>>` | Component & Subcomponent (`^`, `&`) | `[["A", ["B", "C"]]]` | `\|A^B&C\|` |
| Nested Repetition Array | Repeated field (`~`) | `[["VALUE1"], ["VALUE2"]]` | `\|VALUE1~VALUE2\|` |

> **Note on `MSH` Segment:**  
> For the `MSH` header segment:
> - Field index 0 (`fields[0]`) specifies the field separator (`"|"`).
> - Field index 1 (`fields[1]`) specifies encoding characters (`"^~\\&"`).
> - Subsequent fields map to MSH-3 (`Sending Application`), MSH-4 (`Sending Facility`), MSH-9 (`Message Type`), etc.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data containing structured HL7 JSON object if `data` parameter is omitted. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when HL7 generation succeeds. Contains `result` (the raw HL7 text string) and `format` (`"HL7"`). |
| `error` | `Error` | Emitted when HL7 data structure is missing or generator fails. |

---

### Output Object Structure

```json
{
  "result": "MSH|^~\\&|HIS|HOSPITAL|RIS|RADIOLOGY|20260810153000||ADT^A01|MSG00001|P|2.5\rEVN|A01|20260810153000\rPID|1||PAT12345||DOE^JOHN^A||19800101|M\rPV1|1|I|FLAG^200^1",
  "format": "HL7"
}
```

| Property | Type | Description |
|----------|------|-------------|
| `result` | `string` | Complete HL7 v2.x message string with carriage-return (`\r`) segment line breaks. |
| `format` | `string` | Format indicator constant (`"HL7"`). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Generate Patient Admission Message (ADT^A01)

Configure the `data` parameter with a structured JSON object to generate a standard ADT^A01 message.

**Parameter Configuration:**

```json
{
  "delimiters": {
    "field": "|",
    "component": "^",
    "repetition": "~",
    "escape": "\\",
    "subcomponent": "&"
  },
  "segments": [
    {
      "id": "MSH",
      "fields": [
        "|",
        "^~\\&",
        "EPIC",
        "GENERAL_HOSPITAL",
        "PACW",
        "RADIOLOGY",
        "20260810120000",
        "",
        ["ADT", "A01"],
        "MSG20260810001",
        "P",
        "2.5"
      ]
    },
    {
      "id": "EVN",
      "fields": [
        "A01",
        "20260810120000"
      ]
    },
    {
      "id": "PID",
      "fields": [
        "1",
        "",
        "MRN998877",
        "",
        ["SMITH", "ALICE", "M"],
        "",
        "19920515",
        "F"
      ]
    },
    {
      "id": "PV1",
      "fields": [
        "1",
        "I",
        ["EAST", "401", "B"]
      ]
    }
  ]
}
```

**Output `result` String:**

```text
MSH|^~\&|EPIC|GENERAL_HOSPITAL|PACW|RADIOLOGY|20260810120000||ADT^A01|MSG20260810001|P|2.5\rEVN|A01|20260810120000\rPID|1||MRN998877||SMITH^ALICE^M||19920515|F\rPV1|1|I|EAST^401^B
```

---

### Example 2: Dynamic Input from Preceding Node

When no `data` parameter is set in the node configuration, the node automatically reads the JSON payload from the incoming workflow data.

**Workflow Configuration:**

```text
Function Node (builds JSON message structure)
  → EDI Generate HL7 (data: empty)
  → HTTP Request (POST raw HL7 string to Hospital Mirth Engine)
```

**Function Node Code:**

```javascript
return {
  delimiters: {
    field: "|",
    component: "^",
    repetition: "~",
    escape: "\\",
    subcomponent: "&"
  },
  segments: [
    {
      id: "MSH",
      fields: ["|", "^~\\&", "APP", "FAC", "DEST", "DEST_FAC", "20260810150000", "", ["ORU", "R01"], "MSG1001", "P", "2.3.1"]
    },
    {
      id: "OBX",
      fields: ["1", "NM", ["GLU", "Glucose"], "1", "105", "mg/dL", ["70", "110"], "N", "", "", "F"]
    }
  ]
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate HL7 v2.x message from JSON data
```

### Common Patterns

- **API Gateway to HL7:** Receive JSON via Webhook → Function Node → EDI Generate HL7 → Log / Mirth Engine.
- **EHR Data Sync:** Query Database → Format JSON → EDI Generate HL7 → TCP Client / MLLP.
- **Bi-directional EDI Pipeline:** EDI Parse HL7 → Function Node (modify patient record) → EDI Generate HL7.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `HL7 data structure is required`

**Cause:** Neither the node parameter `data` nor the incoming workflow input contained an HL7 JSON object.

**Solution:** Ensure the `data` parameter is filled with a valid object containing `delimiters` and `segments`, or ensure the preceding node returns this structure.

#### Output missing field or component delimiters

**Cause:** The `delimiters` object is incomplete or missing required properties (`field`, `component`, `repetition`, `escape`, `subcomponent`).

**Solution:** Include all 5 delimiter definitions in `delimiters`:

```json
"delimiters": {
  "field": "|",
  "component": "^",
  "repetition": "~",
  "escape": "\\",
  "subcomponent": "&"
}
```

#### Segments joined without line breaks

**Cause:** HL7 v2.x requires carriage return (`\r`) as the segment line break. Standard text editors may display `\r` on a single line.

**Solution:** This is normal behavior for HL7 v2.x. When writing to a file or transmitting via MLLP/TCP, `\r` is preserved correctly.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `HL7 data structure is required` | No input object supplied | Provide valid `delimiters` and `segments` object |
| `HL7 generation failed: Cannot read properties of undefined` | `segments` array is missing or invalid | Ensure `segments` is an array of `{ id, fields }` objects |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [EDI Parse HL7](../edi-parse-hl7/en.md) — Parse HL7 v2.x raw messages into JSON
- [EDI Generate EDIFACT](../edi-generate-edifact/en.md) — Generate UN/EDIFACT documents from JSON
- [EDI Parse EDIFACT](../edi-parse-edifact/en.md) — Parse UN/EDIFACT documents into JSON
- [EDI Generate X12](../edi-generate-x12/en.md) — Generate ANSI ASC X12 documents from JSON
- [EDI Parse X12](../edi-parse-x12/en.md) — Parse ANSI ASC X12 documents into JSON

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->

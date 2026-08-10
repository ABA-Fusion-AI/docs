---
node_id: "edi-idoc-to-json"
title: "IDoc to JSON Converter"
description: "Converts SAP IDoc XML format to JSON. Supports DESADV, SHPMNT, ZFREINV, ZLEITRACK, STPPOD, STATUS, and ZINVRSP message types."
category: "data-transformation-etl"
subcategory: "edi-structured-messages"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - edi
  - sap
  - idoc
  - xml
  - json
  - data-transformation
related_nodes:
  - edi-json-to-idoc
  - edi-parse-edifact
  - edi-parse-x12
---

<!-- SECTION: header -->
# IDoc to JSON Converter

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Convert SAP IDoc XML documents into structured or flat JSON data. The node detects supported IDoc structures, parses the `EDI_DC40` control record, extracts message-specific business data, and optionally validates the IDoc header.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **IDoc to JSON Converter** node converts SAP IDoc XML into JSON.

Internally, the node uses `fast-xml-parser` to parse the XML input, searches recursively for the IDoc structure, reads the `EDI_DC40` control record, detects the IDoc type, and transforms message-specific segments into normalized JSON.

### Supported IDoc Types

The current implementation supports:

- `DESADV`
- `SHPMNT`
- `ZFREINV`
- `ZLEITRACK`
- `STPPOD`
- `STATUS`
- `ZINVRSP`

### Key Features

- **SAP IDoc XML Parsing:** Converts XML IDoc content to JSON.
- **Flexible Input Detection:** Accepts XML strings, JSON strings containing XML, or objects containing XML data.
- **Recursive Input Search:** Searches nested objects for common XML properties.
- **Automatic IDoc Detection:** Detects the supported IDoc type from the parsed XML structure and message type.
- **EDI_DC40 Header Parsing:** Extracts standard sender, receiver, control, date, time, and document information.
- **Structured Output:** Returns normalized header and message-specific data.
- **Flat Output:** Can flatten nested JSON into dotted property paths.
- **Optional Empty Fields:** Can include empty collections and structures when configured.
- **Optional Structure Validation:** Can reject IDocs missing both `IDOCTYP` and `MESTYP`.
- **Multiple SAP Message Types:** Supports logistics, shipment, invoice, tracking, proof-of-delivery, status, and invoice-response messages.

### General Processing Flow

```text
Input XML / JSON / Object
        ↓
Extract XML content
        ↓
Normalize XML
        ↓
Parse XML
        ↓
Find IDOC structure
        ↓
Parse EDI_DC40 header
        ↓
Detect IDoc type
        ↓
Parse message-specific data
        ↓
Structured or Flat JSON
```

### Use Cases

- Convert SAP IDoc XML into application-friendly JSON
- Process delivery notifications
- Parse shipment messages
- Convert freight invoice IDocs
- Process tracking events
- Parse proof-of-delivery information
- Read SAP IDoc status messages
- Process invoice response messages
- Prepare SAP IDoc data for downstream workflow nodes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputData` | `string` | ❌ No | — | SAP IDoc XML configured directly in the node. |
| `outputFormat` | `string` | ❌ No | `structured` | Output format. Supported values: `structured` and `flat`. |
| `includeEmptyFields` | `boolean` | ❌ No | `false` | Includes optional empty collections or structures in supported message outputs. |
| `validateStructure` | `boolean` | ❌ No | `true` | Validates that the parsed IDoc header contains `IDOCTYP` or `MESTYP`. |

### Input Priority

The node first uses configured `inputData` when it is available.

If no configured input is provided, the node can extract XML from the incoming workflow data.

### Supported Incoming Input Types

#### Direct XML String

```xml
<IDOC>
  ...
</IDOC>
```

#### JSON String Containing XML

The node attempts to parse strings beginning with `{` or `[` as JSON and recursively searches the parsed value for XML.

#### Object Containing XML

The node searches these properties:

```text
data
xml
idoc
content
text
xmlData
body
payload
```

The search is recursive when the property contains another object.

### XML Normalization

Before parsing, the node:

- removes a UTF-8 BOM when present;
- trims leading and trailing whitespace;
- converts `CRLF` line endings to `LF`;
- converts remaining `CR` characters to `LF`.

### Output Format

#### Structured

With:

```text
OutputFormat = structured
```

the node returns:

```text
idocType
messageType
header
data
```

#### Flat

With:

```text
OutputFormat = flat
```

the node flattens the header and parsed business data.

Header properties use paths such as:

```text
header.tabnam
header.idoctyp
header.mestyp
```

Nested data properties use paths such as:

```text
data.header.do_number
data.items[0].material_number
```

### Include Empty Fields

When `includeEmptyFields` is `false`, optional collections are only added when corresponding IDoc segments exist.

When enabled, supported parsers may include those properties even when no matching data is available.

This behavior is message-type specific.

### Structure Validation

When `validateStructure` is enabled, the node checks:

```text
header.idoctyp
header.mestyp
```

If both are empty, execution fails with:

```text
IDoc parsing failed: Invalid IDoc: Missing header information
```

The node does not perform full SAP schema validation.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `inputData` | `string` | Configured XML string. |
| `input` | `unknown` | Incoming workflow value used when `inputData` is not configured. |

Incoming workflow input can be:

- XML text;
- JSON text containing XML;
- an object containing XML in one of the supported property names.

### Structured Output

A successful structured response has this form:

```json
{
  "idocType": "DESADV",
  "messageType": "DESADV",
  "header": {},
  "data": {}
}
```

### Header Structure

The `header` object is parsed from `EDI_DC40`.

| Field | Source |
|-------|--------|
| `tabnam` | `TABNAM` |
| `direct` | `DIRECT` |
| `idoctyp` | `IDOCTYP` |
| `mestyp` | `MESTYP` |
| `sndpor` | `SNDPOR` |
| `sndprt` | `SNDPRT` |
| `sndprn` | `SNDPRN` |
| `rcvpor` | `RCVPOR` |
| `rcvprt` | `RCVPRT` |
| `rcvprn` | `RCVPRN` |
| `credat` | `CREDAT` |
| `cretim` | `CRETIM` |
| `docnum` | `DOCNUM` |
| `serial` | `SERIAL` |
| `mandt` | `MANDT` |

The parser also supports these values when they are represented as XML attributes using the `@_` prefix produced by `fast-xml-parser`.

### Tested DESADV Output

The tested XML produced a result equivalent to:

```json
{
  "idocType": "DESADV",
  "messageType": "DESADV",
  "header": {
    "tabnam": "EDI_DC40",
    "direct": 2,
    "idoctyp": "DELVRY03",
    "mestyp": "DESADV",
    "sndpor": "SAPFUSION",
    "sndprt": "LS",
    "sndprn": "FUSIONAI",
    "rcvpor": "SAPCLIENT",
    "rcvprt": "LS",
    "rcvprn": "CLIENT01",
    "credat": 20260810,
    "cretim": 114000,
    "docnum": 1,
    "serial": 20260810114000,
    "mandt": ""
  },
  "data": {
    "header": {
      "do_number": "0080001234",
      "delivery_mode": "1000",
      "sales_organization": "1000",
      "delivery_type": "LF",
      "route": "R001",
      "shipping_conditions": "01",
      "net_weight": "10.50",
      "gross_weight": "12.00",
      "weight_unit": "KG",
      "volume": "0.50",
      "volume_unit": "M3",
      "total_packages": "1"
    },
    "items": [
      {
        "position_number": "000010",
        "material_number": "MAT-001",
        "short_text": "Test Product",
        "plant": "1000",
        "storage_location": "0001",
        "qty_to_deliver": "5",
        "sales_unit": "EA",
        "base_unit": "EA",
        "ean": "1234567890123"
      }
    ]
  }
}
```

> Depending on how `fast-xml-parser` interprets numeric-looking XML values, some output values can be numbers instead of strings.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: supported-types -->
## Supported IDoc Types

### DESADV

Detected when:

- `E1EDL20` exists;
- `MESTYP` is `DESADV`; and
- `E1EDL20.PODAT` is not present.

If `PODAT` is present, the current implementation classifies the structure as `STPPOD`.

The parser extracts:

- delivery header information;
- addresses from `E1ADRM1`;
- delivery items from `E1EDL24`;
- deadlines from `E1EDT13`.

### STPPOD

Detected when:

- `E1EDL20` exists; and
- `MESTYP` is `STPPOD`;

or when `MESTYP` is `DESADV` and `E1EDL20.PODAT` is present.

The parser extracts:

- delivery order number;
- proof-of-delivery date and time;
- delivery items;
- delivery quantity information;
- difference reasons from `E1EDL53`.

### SHPMNT

Detected when:

```text
E1EDT20
```

exists.

The parser extracts:

- shipment number;
- shipment type;
- gross weight and volume from text lines when available;
- transport mode;
- delivery order numbers;
- handling-unit information.

### ZFREINV

Detected when:

```text
Z1FREINVH
```

exists and `MESTYP` is not `ZINVRSP`.

The parser extracts:

- invoice header information;
- message function code;
- invoice items and amounts.

### ZINVRSP

Detected when:

```text
Z1FREINVH
```

exists and:

```text
MESTYP = ZINVRSP
```

The parser extracts:

- referenced document number;
- invoice number;
- response texts.

### ZLEITRACK

Detected when:

```text
Z1TRKH
```

exists.

The parser extracts:

- delivery order;
- event code;
- event date and time;
- description;
- partner identifier.

### STATUS

Detected when:

```text
E1STATS
```

exists.

The parser extracts:

- document number;
- status;
- status text;
- log date and time;
- username;
- program name;
- route identifier;
- status code.

<!-- /SECTION: supported-types -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: DESADV

The following minimal structure was successfully tested:

```xml
<IDOC>
  <EDI_DC40>
    <TABNAM>EDI_DC40</TABNAM>
    <DIRECT>2</DIRECT>
    <IDOCTYP>DELVRY03</IDOCTYP>
    <MESTYP>DESADV</MESTYP>
    <SNDPOR>SAPFUSION</SNDPOR>
    <SNDPRT>LS</SNDPRT>
    <SNDPRN>FUSIONAI</SNDPRN>
    <RCVPOR>SAPCLIENT</RCVPOR>
    <RCVPRT>LS</RCVPRT>
    <RCVPRN>CLIENT01</RCVPRN>
    <CREDAT>20260810</CREDAT>
    <CRETIM>114000</CRETIM>
    <DOCNUM>0000000000000001</DOCNUM>
    <SERIAL>20260810114000</SERIAL>
  </EDI_DC40>

  <E1EDL20>
    <VBELN>0080001234</VBELN>
    <VSTEL>1000</VSTEL>
    <VKORG>1000</VKORG>
    <ROUTE>R001</ROUTE>
    <VSBED>01</VSBED>

    <E1EDL21>
      <LFART>LF</LFART>
    </E1EDL21>

    <E1EDL24>
      <POSNR>000010</POSNR>
      <MATNR>MAT-001</MATNR>
      <ARKTX>Test Product</ARKTX>
      <WERKS>1000</WERKS>
      <LGORT>0001</LGORT>
      <LFIMG>5</LFIMG>
      <VRKME>EA</VRKME>
      <MEINS>EA</MEINS>
      <EAN11>1234567890123</EAN11>
    </E1EDL24>
  </E1EDL20>
</IDOC>
```

Configuration:

```text
OutputFormat       = structured
IncludeEmptyFields = false
ValidateStructure  = true
```

The result is detected as:

```text
idocType    = DESADV
messageType = DESADV
```

---

### Example: Flat Output

With:

```text
OutputFormat = flat
```

a structured value such as:

```json
{
  "header": {
    "mestyp": "DESADV"
  },
  "data": {
    "header": {
      "do_number": "0080001234"
    }
  }
}
```

is flattened into properties equivalent to:

```json
{
  "header.mestyp": "DESADV",
  "data.header.do_number": "0080001234"
}
```

---

### Example: Incoming Object

When `inputData` is not configured:

```json
{
  "xml": "<IDOC>...</IDOC>"
}
```

can be used as incoming workflow data.

The following keys are recognized:

```text
data
xml
idoc
content
text
xmlData
body
payload
```

---

### Example: JSON String Containing XML

An incoming JSON string can contain an XML value:

```json
{
  "payload": "<IDOC>...</IDOC>"
}
```

The node parses the JSON first and then recursively searches for the XML value.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert a SAP IDoc to JSON
```

### Common Patterns

- **Parse IDoc:** Manual Trigger → IDoc to JSON Converter → Log
- **Convert DESADV:** Manual Trigger → IDoc to JSON Converter → Log
- **Flatten IDoc:** Manual Trigger → IDoc to JSON Converter → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### "No valid XML input data found"

**Cause**

The node could not find a usable XML string in either the configured input or incoming workflow data.

**Solution**

Provide:

- an XML string;
- a JSON string containing XML;
- or an object containing XML in a supported property.

The full wrapped error is:

```text
IDoc parsing failed: No valid XML input data found. Expected XML string, JSON string with XML, or object with XML data.
```

---

### "Unknown IDoc structure"

**Cause**

The parsed XML does not contain one of the supported IDoc segment structures.

The detector expects one of:

```text
E1EDL20
E1EDT20
Z1FREINVH
Z1TRKH
E1STATS
```

**Solution**

Verify that the XML contains a supported SAP IDoc structure.

---

### Empty Message Type

If `EDI_DC40.MESTYP` is missing, the parser can report:

```text
MessageType: empty
```

Some IDoc types such as `SHPMNT`, `ZFREINV`, `ZLEITRACK`, and `STATUS` can still be detected from their segment structure. `DESADV` and `STPPOD`, however, require a recognized `MESTYP` together with `E1EDL20` in the current implementation.

---

### "Invalid IDoc: Missing header information"

**Cause**

`validateStructure` is enabled and both:

```text
header.idoctyp
header.mestyp
```

are empty.

**Solution**

Provide a valid `EDI_DC40` header containing at least `IDOCTYP` or `MESTYP`.

---

### Direct `<IDOC>` Input

The tested workflow succeeded using:

```xml
<IDOC>
  ...
</IDOC>
```

This structure is directly recognized by the recursive IDoc search.

If an outer wrapper is used, the parser recursively searches nested objects up to the implementation's recursion limit.

---

### Output Format

If `outputFormat` is:

```text
structured
```

the node returns:

```text
idocType
messageType
header
data
```

If it is:

```text
flat
```

the node returns flattened header and data fields.

The top-level `idocType` and `messageType` values are not added separately by the flat conversion method.

---

### Error Handling

All execution errors are wrapped as:

```text
IDoc parsing failed: <error message>
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EDI JSON to IDoc
- EDI Parse EDIFACT
- EDI Parse X12

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-10 | Initial release of the IDoc to JSON Converter documentation. |

<!-- /SECTION: changelog -->
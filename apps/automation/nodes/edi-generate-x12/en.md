---
node_id: "edi-generate-x12"
title: "EDI Generate X12"
description: "Generate ANSI ASC X12 documents from structured JSON data."
category: "data-transformation-etl"
subcategory: "edi-structured-messages"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - edi
  - x12
  - ansi-x12
  - b2b
  - integration
  - structured-data
related_nodes:
  - edi-parse-x12
  - edi-generate-edifact
  - edi-parse-edifact
---

<!-- SECTION: header -->
# EDI Generate X12

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Generate ANSI ASC X12 documents from structured JSON data. The node converts delimiters, interchange information, functional groups, transaction sets, and business segments into an X12-formatted string.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EDI Generate X12** node generates an ANSI ASC X12 document from a structured JSON object.

Internally, the node uses `X12Parser.generate()` to construct the X12 interchange. The generated document contains an ISA interchange header, one or more functional groups, transaction sets, business segments, transaction trailers, group trailers, and an IEA interchange trailer.

### Key Features

- **Structured JSON Input:** Generates X12 from structured interchange and transaction data.
- **Custom Delimiters:** Supports configurable element, segment, and component delimiters.
- **Interchange Envelope:** Generates `ISA` and `IEA` segments.
- **Functional Groups:** Generates `GS` and `GE` segments.
- **Transaction Sets:** Generates `ST` and `SE` segments.
- **Business Segments:** Supports custom X12 segment tags and elements.
- **Composite Elements:** Converts array values into component-separated values.
- **Multiple Functional Groups:** Supports multiple groups in one interchange.
- **Multiple Transactions:** Supports multiple transaction sets inside each functional group.
- **Automatic Group Count:** Uses `functionalGroups.length` when generating the `IEA` segment.
- **Plain Text Output:** Returns the generated X12 document as a string.

### Generated Structure

The generator produces a document in this general order:

```text
ISA
GS
ST
Business segments
SE
GE
IEA
```

With multiple transactions:

```text
ISA
GS
ST
Business segments
SE
ST
Business segments
SE
GE
IEA
```

With multiple functional groups:

```text
ISA
GS
ST
...
SE
GE
GS
ST
...
SE
GE
IEA
```

### Delimiters

The generator reads the delimiters from the input object.

The tested configuration uses:

| Purpose | Character |
|---------|-----------|
| Element delimiter | `*` |
| Segment delimiter | `~` |
| Component delimiter | `:` |

### Use Cases

- Generate ANSI X12 invoices
- Create X12 transaction sets from application data
- Build B2B integration workflows
- Generate supply-chain documents
- Prepare X12 payloads for trading partners
- Convert structured business data into X12 syntax
- Integrate ERP and EDI systems

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `any` | ✅ Yes | — | Structured X12 data. When not configured, the node attempts to use the incoming workflow object. |

Although the schema accepts `any`, the generator expects the value to follow the structure described below.

### Root Structure

The generator expects:

```json
{
  "delimiters": {
    "element": "*",
    "segment": "~",
    "component": ":"
  },
  "interchange": {},
  "functionalGroups": []
}
```

### Delimiters Structure

| Field | Expected Type | Description |
|-------|---------------|-------------|
| `element` | `string` | Separator used between X12 data elements. |
| `segment` | `string` | Terminator used after each X12 segment. |
| `component` | `string` | Separator used inside composite elements. |

Example:

```json
{
  "delimiters": {
    "element": "*",
    "segment": "~",
    "component": ":"
  }
}
```

### Interchange Structure

The `interchange` object is used when generating the `ISA` and `IEA` segments.

| Field | Expected Type | Used By | Description |
|-------|---------------|---------|-------------|
| `authorizationQualifier` | `string` | `ISA` | Authorization information qualifier. |
| `authorizationInformation` | `string` | `ISA` | Authorization information. |
| `securityQualifier` | `string` | `ISA` | Security information qualifier. |
| `securityInformation` | `string` | `ISA` | Security information. |
| `senderQualifier` | `string` | `ISA` | Interchange sender qualifier. |
| `senderId` | `string` | `ISA` | Sender identifier. |
| `receiverQualifier` | `string` | `ISA` | Interchange receiver qualifier. |
| `receiverId` | `string` | `ISA` | Receiver identifier. |
| `date` | `string` | `ISA` | Interchange date. |
| `time` | `string` | `ISA` | Interchange time. |
| `standardsId` | `string` | `ISA` | Interchange standards identifier. |
| `versionNumber` | `string` | `ISA` | Interchange control version number. |
| `controlNumber` | `string` | `ISA`, `IEA` | Interchange control number. |
| `acknowledgmentRequested` | `string` | `ISA` | Acknowledgment requested flag. |
| `usageIndicator` | `string` | `ISA` | Test or production usage indicator. |

Example:

```json
{
  "interchange": {
    "authorizationQualifier": "00",
    "authorizationInformation": "",
    "securityQualifier": "00",
    "securityInformation": "",
    "senderQualifier": "ZZ",
    "senderId": "FUSIONAI",
    "receiverQualifier": "ZZ",
    "receiverId": "CLIENT01",
    "date": "260807",
    "time": "1430",
    "standardsId": "U",
    "versionNumber": "00401",
    "controlNumber": "1",
    "acknowledgmentRequested": "0",
    "usageIndicator": "T"
  }
}
```

### ISA Padding Behavior

The generator applies fixed-width padding to several ISA fields:

- `authorizationQualifier` → padded to 2 characters
- `authorizationInformation` → padded to 10 characters
- `securityQualifier` → padded to 2 characters
- `securityInformation` → padded to 10 characters
- `senderId` → padded to 15 characters
- `receiverId` → padded to 15 characters
- `controlNumber` → left-padded to 9 characters

For example:

```json
"controlNumber": "1"
```

is generated as:

```text
000000001
```

### Functional Groups Structure

The `functionalGroups` property contains one or more functional groups.

Each group contains:

| Field | Expected Type | Description |
|-------|---------------|-------------|
| `header` | `object` | GS segment information. |
| `transactions` | `array` | Transaction sets contained in the group. |
| `transactionCount` | `number` | Value written into the GE segment. |
| `controlNumber` | `string` | Functional group control number written into GE. |

### Functional Group Header

The `header` object contains:

| Field | Expected Type | Used By |
|-------|---------------|---------|
| `functionalIdentifierCode` | `string` | `GS` |
| `applicationSenderCode` | `string` | `GS` |
| `applicationReceiverCode` | `string` | `GS` |
| `date` | `string` | `GS` |
| `time` | `string` | `GS` |
| `groupControlNumber` | `string` | `GS` |
| `responsibleAgencyCode` | `string` | `GS` |
| `versionReleaseCode` | `string` | `GS` |

Example:

```json
{
  "header": {
    "functionalIdentifierCode": "IN",
    "applicationSenderCode": "FUSIONAI",
    "applicationReceiverCode": "CLIENT01",
    "date": "20260807",
    "time": "1430",
    "groupControlNumber": "1",
    "responsibleAgencyCode": "X",
    "versionReleaseCode": "004010"
  }
}
```

### Transaction Structure

Each transaction contains:

| Field | Expected Type | Description |
|-------|---------------|-------------|
| `header` | `object` | ST segment information. |
| `segments` | `array` | Business segments generated between ST and SE. |
| `segmentCount` | `number` | Segment count written to SE. |
| `controlNumber` | `string` | Control number written to SE. |

> The generator does not validate that `header.transactionSetControlNumber` and `controlNumber` match. Use the same transaction control number in `ST` and `SE` when required by your X12 document.

### Transaction Header

The transaction header contains:

| Field | Expected Type | Description |
|-------|---------------|-------------|
| `transactionSetIdentifierCode` | `string` | X12 transaction set identifier, such as `810`. |
| `transactionSetControlNumber` | `string` | ST control number. |
| `implementationConventionReference` | `string` | Optional implementation convention reference. |

Example:

```json
{
  "header": {
    "transactionSetIdentifierCode": "810",
    "transactionSetControlNumber": "0001"
  }
}
```

### Segment Structure

Each business segment contains:

| Field | Expected Type | Description |
|-------|---------------|-------------|
| `tag` | `string` | X12 segment identifier such as `BIG`, `N1`, or `REF`. |
| `elements` | `array` | Segment data elements. |

Example:

```json
{
  "tag": "BIG",
  "elements": [
    "20260807",
    "INV1001"
  ]
}
```

This generates:

```text
BIG*20260807*INV1001~
```

### Composite Elements

When a segment element is an array, the generator joins its values using the configured component delimiter.

For example:

```json
["A", "B", "C"]
```

with:

```json
"component": ":"
```

becomes:

```text
A:B:C
```

### Segment Count

The generator writes the provided `segmentCount` directly into the `SE` segment.

For the tested transaction:

```text
ST
BIG
SE
```

the value is:

```json
"segmentCount": 3
```

The generation method itself does not recalculate or validate this value.

### Transaction Count

The generator writes `transactionCount` directly into `GE`.

The generation method itself does not recalculate this value from `transactions.length`.

### Functional Group Count

The `IEA` functional group count is generated automatically using:

```text
data.functionalGroups.length
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `data` | `any` | Structured X12 data configured in the node. |
| `input` | `object` | Incoming workflow object used when no configured `data` value is available. |

The node first uses the configured `data` value. If no configured value is available and incoming workflow data is an object, the incoming object is used instead.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `result` | `string` | Generated ANSI ASC X12 document. |
| `format` | `string` | Output format identifier. Always `X12` on successful generation. |

### Successful Response

A successful execution returns:

```json
{
  "result": "ISA*...~GS*...~ST*...~SE*...~GE*...~IEA*...~",
  "format": "X12"
}
```

The exact generated document depends on the supplied delimiters, interchange, functional groups, transactions, segments, and elements.

### Tested Input

The following structure was successfully tested:

```json
{
  "delimiters": {
    "element": "*",
    "segment": "~",
    "component": ":"
  },
  "interchange": {
    "authorizationQualifier": "00",
    "authorizationInformation": "",
    "securityQualifier": "00",
    "securityInformation": "",
    "senderQualifier": "ZZ",
    "senderId": "FUSIONAI",
    "receiverQualifier": "ZZ",
    "receiverId": "CLIENT01",
    "date": "260807",
    "time": "1430",
    "standardsId": "U",
    "versionNumber": "00401",
    "controlNumber": "1",
    "acknowledgmentRequested": "0",
    "usageIndicator": "T"
  },
  "functionalGroups": [
    {
      "header": {
        "functionalIdentifierCode": "IN",
        "applicationSenderCode": "FUSIONAI",
        "applicationReceiverCode": "CLIENT01",
        "date": "20260807",
        "time": "1430",
        "groupControlNumber": "1",
        "responsibleAgencyCode": "X",
        "versionReleaseCode": "004010"
      },
      "transactionCount": 1,
      "controlNumber": "1",
      "transactions": [
        {
          "header": {
            "transactionSetIdentifierCode": "810",
            "transactionSetControlNumber": "0001"
          },
          "segmentCount": 3,
          "controlNumber": "0001",
          "segments": [
            {
              "tag": "BIG",
              "elements": [
                "20260807",
                "INV1001"
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

### Generated Structure

The tested input generates sections equivalent to:

```text
ISA*...
GS*IN*FUSIONAI*CLIENT01*20260807*1430*1*X*004010~
ST*810*0001~
BIG*20260807*INV1001~
SE*3*0001~
GE*1*1~
IEA*1*000000001~
```

### Error Response

Errors raised during generation are wrapped with:

```text
X12 generation failed: <error message>
```

For example, missing delimiters can lead to an error such as:

```text
X12 generation failed: Cannot read properties of undefined (reading 'element')
```

Missing interchange data can also produce an error related to properties used by the ISA generator.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Generate an X12 810 Invoice

**Configuration**

```json
{
  "delimiters": {
    "element": "*",
    "segment": "~",
    "component": ":"
  },
  "interchange": {
    "authorizationQualifier": "00",
    "authorizationInformation": "",
    "securityQualifier": "00",
    "securityInformation": "",
    "senderQualifier": "ZZ",
    "senderId": "FUSIONAI",
    "receiverQualifier": "ZZ",
    "receiverId": "CLIENT01",
    "date": "260807",
    "time": "1430",
    "standardsId": "U",
    "versionNumber": "00401",
    "controlNumber": "1",
    "acknowledgmentRequested": "0",
    "usageIndicator": "T"
  },
  "functionalGroups": [
    {
      "header": {
        "functionalIdentifierCode": "IN",
        "applicationSenderCode": "FUSIONAI",
        "applicationReceiverCode": "CLIENT01",
        "date": "20260807",
        "time": "1430",
        "groupControlNumber": "1",
        "responsibleAgencyCode": "X",
        "versionReleaseCode": "004010"
      },
      "transactionCount": 1,
      "controlNumber": "1",
      "transactions": [
        {
          "header": {
            "transactionSetIdentifierCode": "810",
            "transactionSetControlNumber": "0001"
          },
          "segmentCount": 3,
          "controlNumber": "0001",
          "segments": [
            {
              "tag": "BIG",
              "elements": [
                "20260807",
                "INV1001"
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

The generated document contains:

```text
ISA
GS
ST
BIG
SE
GE
IEA
```

---

### Example: Add Another Business Segment

Add a reference segment:

```json
{
  "header": {
    "transactionSetIdentifierCode": "810",
    "transactionSetControlNumber": "0001"
  },
  "segmentCount": 4,
  "controlNumber": "0001",
  "segments": [
    {
      "tag": "BIG",
      "elements": [
        "20260807",
        "INV1001"
      ]
    },
    {
      "tag": "REF",
      "elements": [
        "IA",
        "CUSTOMER01"
      ]
    }
  ]
}
```

The generated transaction contains:

```text
ST*810*0001~
BIG*20260807*INV1001~
REF*IA*CUSTOMER01~
SE*4*0001~
```

---

### Example: Composite Segment Element

Given:

```json
{
  "tag": "REF",
  "elements": [
    ["A", "B", "C"]
  ]
}
```

and:

```json
"component": ":"
```

the generated segment is:

```text
REF*A:B:C~
```

---

### Example: Multiple Transactions

A functional group can contain more than one transaction:

```json
{
  "transactionCount": 2,
  "controlNumber": "1",
  "transactions": [
    {
      "header": {
        "transactionSetIdentifierCode": "810",
        "transactionSetControlNumber": "0001"
      },
      "segmentCount": 3,
      "controlNumber": "0001",
      "segments": [
        {
          "tag": "BIG",
          "elements": ["20260807", "INV1001"]
        }
      ]
    },
    {
      "header": {
        "transactionSetIdentifierCode": "810",
        "transactionSetControlNumber": "0002"
      },
      "segmentCount": 3,
      "controlNumber": "0002",
      "segments": [
        {
          "tag": "BIG",
          "elements": ["20260807", "INV1002"]
        }
      ]
    }
  ]
}
```

The `GE` segment should use:

```text
2
```

for the transaction count when `transactionCount` is configured as `2`.

---

### Example: Multiple Functional Groups

When multiple entries are supplied in `functionalGroups`, the generator produces a `GS`/`GE` envelope for each group.

The `IEA` group count is calculated automatically from:

```text
functionalGroups.length
```

---

### Example: Optional Implementation Convention Reference

When:

```json
{
  "transactionSetIdentifierCode": "810",
  "transactionSetControlNumber": "0001",
  "implementationConventionReference": "004010"
}
```

the generator appends the value to the `ST` segment.

---

### Example: Missing Data

If no usable X12 structure is provided, the node throws:

```text
X12 generation failed: X12 data structure is required
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate an ANSI X12 Document
```

### Common Patterns

- **Generate X12:** Manual Trigger → EDI Generate X12 → Log
- **Invoice Generation:** Manual Trigger → EDI Generate X12 → Log
- **Multiple Transaction Generation:** Manual Trigger → EDI Generate X12 → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "X12 data structure is required"

**Cause**

No X12 data was provided to the node.

**Solution**

Provide a structured object containing:

```json
{
  "delimiters": {},
  "interchange": {},
  "functionalGroups": []
}
```

For a meaningful X12 document, populate the delimiter, ISA, functional group, and transaction fields expected by the generator.

---

#### "Cannot read properties of undefined (reading 'element')"

**Cause**

The `delimiters` object is missing.

The generator attempts to access:

```text
data.delimiters.element
```

**Solution**

Provide:

```json
{
  "delimiters": {
    "element": "*",
    "segment": "~",
    "component": ":"
  }
}
```

---

#### Missing Interchange Data

**Cause**

The `interchange` object is missing or incomplete.

The generator reads ISA values directly from:

```text
data.interchange
```

**Solution**

Provide the expected ISA fields, including sender, receiver, dates, version information, and control number.

---

#### Incorrect Transaction Segment Count

**Cause**

The supplied `segmentCount` does not match the intended transaction structure.

The generator writes this value directly into `SE`.

**Solution**

Count:

- `ST`
- every business segment
- `SE`

For one business segment:

```text
ST
BIG
SE
```

use:

```json
"segmentCount": 3
```

For two business segments:

```text
ST
BIG
REF
SE
```

use:

```json
"segmentCount": 4
```

---

#### Incorrect Functional Group Transaction Count

**Cause**

The configured `transactionCount` does not match the intended number of transactions.

The generator writes this value directly into `GE`.

**Solution**

Set `transactionCount` to the desired number of transaction sets in the functional group.

---

#### Control Number Formatting

**Cause**

The ISA control number appears with additional leading zeros.

**Explanation**

The generator automatically left-pads `interchange.controlNumber` to nine characters.

For example:

```json
"controlNumber": "1"
```

becomes:

```text
000000001
```

---

#### Sender or Receiver Padding

**Cause**

The generated ISA sender or receiver contains trailing spaces.

**Explanation**

The generator pads both values to 15 characters as required by its current implementation.

---

#### Unexpected Composite Formatting

**Cause**

An array was used as a segment element.

**Explanation**

Array values are joined using the configured component delimiter.

For example:

```json
["A", "B", "C"]
```

with:

```text
:
```

becomes:

```text
A:B:C
```

---

### Error Messages

| Error | Description |
|-------|-------------|
| `X12 generation failed: X12 data structure is required` | No usable X12 input data was provided. |
| `X12 generation failed: Cannot read properties of undefined (reading 'element')` | The required `delimiters` object is missing. |
| `X12 generation failed: <error message>` | The node wrapped an error raised while generating the X12 document. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EDI Parse X12
- EDI Generate EDIFACT
- EDI Parse EDIFACT

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial release of the EDI Generate X12 documentation. |

<!-- /SECTION: changelog -->
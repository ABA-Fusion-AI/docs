---
node_id: "edi-parse-x12"
title: "EDI Parse X12"
description: "Parse ANSI ASC X12 documents with delimiter detection and optional validation."
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
  - parsing
  - validation
  - structured-data
related_nodes:
  - edi-generate-x12
  - edi-parse-edifact
  - edi-generate-edifact
---

<!-- SECTION: header -->
# EDI Parse X12

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Parse ANSI ASC X12 documents into structured data. The node detects X12 delimiters from the ISA segment, extracts interchange, functional group, transaction, and business segment information, and optionally validates the parsed document.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EDI Parse X12** node converts an ANSI ASC X12 document into structured workflow data.

Internally, the node uses `X12Parser.parse()` to detect the document delimiters and parse the X12 envelope structure:

```text
ISA
GS
ST
Business segments
SE
GE
IEA
```

When validation is enabled, the node also runs `X12Parser.validate()` and appends the validation result to the parsed output.

### Key Features

- **ANSI ASC X12 Parsing:** Converts X12 documents into structured objects.
- **Delimiter Detection:** Detects element, component, and segment delimiters from the ISA segment.
- **Interchange Parsing:** Extracts ISA and IEA information.
- **Functional Group Parsing:** Extracts GS and GE information.
- **Transaction Parsing:** Extracts ST and SE information.
- **Business Segment Parsing:** Captures transaction segments between ST and SE.
- **Composite Elements:** Splits composite elements using the detected component delimiter.
- **Multiple Groups:** Supports multiple functional groups.
- **Multiple Transactions:** Supports multiple transaction sets within a functional group.
- **Optional Validation:** Checks ISA control number length, functional group transaction counts, and transaction segment counts.
- **Flexible Input Source:** Supports configured input or incoming workflow data.

### Parsed Structure

A parsed X12 document follows this general structure:

```text
standard
delimiters
interchange
functionalGroups
validation       ← when Validate is enabled
```

Each functional group can contain one or more transactions, and each transaction contains its parsed business segments.

### X12 Delimiter Detection

The current parser reads delimiters directly from fixed positions in the ISA segment:

```text
Element delimiter   → character at index 3
Component delimiter → character at index 104
Segment delimiter   → character at index 105
```

Because delimiter detection relies on fixed ISA positions, preserving the required spacing and field widths in the ISA segment is important.

### Use Cases

- Parse ANSI X12 invoices
- Read X12 messages from trading partners
- Extract interchange and transaction metadata
- Inspect business segments
- Validate transaction and functional group counts
- Process B2B integration messages
- Convert X12 payloads into workflow-friendly structured data

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputData` | `string` | ✅ Yes | — | ANSI ASC X12 document to parse. |
| `validate` | `boolean` | ❌ No | `true` | Runs X12-specific validation after parsing. |

### Input Data

A valid example:

```text
ISA*00*          *00*          *ZZ*FUSIONAI       *ZZ*CLIENT01       *260807*1430*U*00401*000000001*0*T*:~GS*IN*FUSIONAI*CLIENT01*20260807*1430*1*X*004010~ST*810*0001~BIG*20260807*INV1001~SE*3*0001~GE*1*1~IEA*1*000000001~
```

### Input Source Behavior

The node first uses the configured `inputData`.

If `inputData` is empty, the node can use incoming workflow data:

- when the incoming value itself is a string;
- when the incoming value is an object containing a `data` property.

Example incoming object:

```json
{
  "data": "ISA*00*...~IEA*1*000000001~"
}
```

If no usable input exists, the node throws:

```text
X12 parsing failed: X12 input data is required
```

### Validation

When:

```text
Validate: true
```

the node:

1. parses the X12 document;
2. runs `parser.validate(parsed)`;
3. returns the parsed structure together with the validation result.

When:

```text
Validate: false
```

the node returns only the parsed X12 structure.

### ISA Formatting Requirement

The parser detects the delimiters using fixed character positions within the ISA segment.

For this reason, values such as:

```text
authorizationInformation
securityInformation
senderId
receiverId
```

must preserve their expected padding when using a standard fixed-length ISA segment.

For example:

```text
ISA*00*          *00*          *ZZ*FUSIONAI       *ZZ*CLIENT01       *...
```

Removing the spaces can shift the delimiter positions and cause incorrect parsing.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `inputData` | `string` | X12 document configured directly in the node. |
| `input` | `string \| object` | Incoming workflow data used when configured `inputData` is empty. |

When the incoming value is an object, the node reads its `data` property.

### Outputs

The parsed output contains:

| Field | Type | Description |
|-------|------|-------------|
| `standard` | `string` | Format identifier. Returns `X12`. |
| `delimiters` | `object` | Detected X12 delimiters. |
| `interchange` | `object` | Parsed ISA and IEA information. |
| `functionalGroups` | `array` | Parsed functional groups and transaction sets. |
| `validation` | `object` | Validation result when `validate` is enabled. |

### Delimiters Structure

The `delimiters` object contains:

| Field | Description |
|-------|-------------|
| `segment` | Segment delimiter detected from the ISA segment. |
| `element` | Data element delimiter. |
| `component` | Composite component delimiter. |

For the tested document:

```json
{
  "segment": "~",
  "element": "*",
  "component": ":"
}
```

### Interchange Structure

The parsed interchange can contain:

| Field | Description |
|-------|-------------|
| `authorizationQualifier` | ISA authorization qualifier. |
| `authorizationInformation` | ISA authorization information. |
| `securityQualifier` | ISA security qualifier. |
| `securityInformation` | ISA security information. |
| `senderQualifier` | Sender qualifier. |
| `senderId` | Sender identifier. |
| `receiverQualifier` | Receiver qualifier. |
| `receiverId` | Receiver identifier. |
| `date` | Interchange date. |
| `time` | Interchange time. |
| `standardsId` | Interchange standards identifier. |
| `versionNumber` | X12 version number. |
| `controlNumber` | ISA interchange control number. |
| `acknowledgmentRequested` | ISA acknowledgment request value. |
| `usageIndicator` | Test or production indicator. |
| `groupCount` | Number of functional groups read from IEA. |
| `controlNumberCheck` | Control number read from IEA. |

### Functional Group Structure

Each item in `functionalGroups` contains:

| Field | Type | Description |
|-------|------|-------------|
| `header` | `object` | Information parsed from GS. |
| `transactions` | `array` | Parsed transaction sets. |
| `transactionCount` | `number` | Transaction count read from GE. |
| `controlNumber` | `string` | Functional group control number read from GE. |

### Functional Group Header

The GS header contains:

| Field | Description |
|-------|-------------|
| `functionalIdentifierCode` | Functional identifier code. |
| `applicationSenderCode` | Application sender code. |
| `applicationReceiverCode` | Application receiver code. |
| `date` | Group date. |
| `time` | Group time. |
| `groupControlNumber` | Functional group control number from GS. |
| `responsibleAgencyCode` | Responsible agency code. |
| `versionReleaseCode` | Version/release identifier. |

### Transaction Structure

Each transaction contains:

| Field | Type | Description |
|-------|------|-------------|
| `header` | `object` | ST header information. |
| `segments` | `array` | Business segments found between ST and SE. |
| `segmentCount` | `number` | Segment count read from SE. |
| `controlNumber` | `string` | Control number read from SE. |

### Transaction Header

The ST header contains:

| Field | Description |
|-------|-------------|
| `transactionSetIdentifierCode` | Transaction set identifier such as `810`. |
| `transactionSetControlNumber` | Transaction set control number. |
| `implementationConventionReference` | Optional implementation convention reference. |

### Segment Structure

Parsed business segments contain:

| Field | Type | Description |
|-------|------|-------------|
| `tag` | `string` | X12 segment identifier. |
| `elements` | `array` | Parsed data elements. |

For example:

```text
BIG*20260807*INV1001~
```

is parsed into a structure equivalent to:

```json
{
  "tag": "BIG",
  "elements": [
    "20260807",
    "INV1001"
  ]
}
```

### Composite Elements

When a segment element contains the detected component delimiter, it is returned as an array.

For example:

```text
A:B:C
```

with component delimiter:

```text
:
```

is parsed as:

```json
[
  "A",
  "B",
  "C"
]
```

### Successful Response

The tested document returns data equivalent to:

```json
{
  "standard": "X12",
  "delimiters": {
    "segment": "~",
    "element": "*",
    "component": ":"
  },
  "interchange": {
    "authorizationQualifier": "00",
    "authorizationInformation": "          ",
    "securityQualifier": "00",
    "securityInformation": "          ",
    "senderQualifier": "ZZ",
    "senderId": "FUSIONAI       ",
    "receiverQualifier": "ZZ",
    "receiverId": "CLIENT01       ",
    "date": "260807",
    "time": "1430",
    "standardsId": "U",
    "versionNumber": "00401",
    "controlNumber": "000000001",
    "acknowledgmentRequested": "0",
    "usageIndicator": "T",
    "groupCount": 1,
    "controlNumberCheck": "000000001"
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
      "transactions": [
        {
          "header": {
            "transactionSetIdentifierCode": "810",
            "transactionSetControlNumber": "0001"
          },
          "segments": [
            {
              "tag": "BIG",
              "elements": [
                "20260807",
                "INV1001"
              ]
            }
          ],
          "segmentCount": 3,
          "controlNumber": "0001"
        }
      ],
      "transactionCount": 1,
      "controlNumber": "1"
    }
  ]
}
```

When validation is enabled, a `validation` property is also returned.

### Error Response

Parsing errors are wrapped as:

```text
X12 parsing failed: <error message>
```

If no usable input exists:

```text
X12 parsing failed: X12 input data is required
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Parse an X12 810 Invoice

**Input Data**

```text
ISA*00*          *00*          *ZZ*FUSIONAI       *ZZ*CLIENT01       *260807*1430*U*00401*000000001*0*T*:~GS*IN*FUSIONAI*CLIENT01*20260807*1430*1*X*004010~ST*810*0001~BIG*20260807*INV1001~SE*3*0001~GE*1*1~IEA*1*000000001~
```

**Configuration**

```text
Validate: true
```

The parser detects:

```text
Element delimiter   = *
Component delimiter = :
Segment delimiter   = ~
```

and extracts one functional group containing one `810` transaction.

---

### Example: Parse Without Validation

```text
Validate: false
```

The node returns the parsed document without adding a `validation` property.

---

### Example: Incoming String

When configured `inputData` is empty, an incoming X12 string can be parsed directly.

Example:

```text
ISA*00*...~IEA*1*000000001~
```

---

### Example: Incoming Object

The node can also read the `data` property from an incoming object:

```json
{
  "data": "ISA*00*...~IEA*1*000000001~"
}
```

---

### Example: Business Segment

Input:

```text
BIG*20260807*INV1001~
```

inside an active transaction becomes:

```json
{
  "tag": "BIG",
  "elements": [
    "20260807",
    "INV1001"
  ]
}
```

---

### Example: Composite Value

If the component delimiter is:

```text
:
```

then:

```text
REF*A:B:C~
```

produces an element equivalent to:

```json
[
  "A",
  "B",
  "C"
]
```

---

### Example: Invalid ISA Length

If padding is removed from the ISA segment, delimiter detection may use incorrect character positions.

Avoid modifying:

```text
ISA*00*          *00*          *ZZ*FUSIONAI       *ZZ*CLIENT01       *...
```

into a shortened form such as:

```text
ISA*00**00**ZZ*FUSIONAI*ZZ*CLIENT01*...
```

because the parser relies on fixed positions for component and segment delimiter detection.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Parse and Validate an ANSI X12 Document
```

### Common Patterns

- **Parse X12:** Manual Trigger → EDI Parse X12 → Log
- **Validate X12:** Manual Trigger → EDI Parse X12 → Log
- **Inspect Transactions:** Manual Trigger → EDI Parse X12 → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "X12 input data is required"

**Cause**

No configured `inputData`, incoming string, or incoming object containing `data` was available.

**Solution**

Provide a valid ANSI ASC X12 document.

---

#### Incorrect Delimiter Detection

**Cause**

The ISA segment does not have the expected fixed layout.

The current parser reads:

```text
data.charAt(3)
data.charAt(104)
data.charAt(105)
```

to detect the element, component, and segment delimiters.

**Solution**

Preserve the required ISA spacing and fixed-length structure.

---

#### ISA Control Number Validation Error

**Cause**

When validation is enabled, the parser requires the ISA `controlNumber` to contain exactly 9 characters.

**Solution**

Use a 9-character interchange control number such as:

```text
000000001
```

---

#### Functional Group Transaction Count Mismatch

**Cause**

The number of parsed transactions does not match the `transactionCount` value read from `GE`.

The validator compares:

```text
group.transactions.length
```

with:

```text
group.transactionCount
```

**Solution**

Ensure the GE transaction count matches the actual number of ST/SE transaction sets in the group.

---

#### Transaction Segment Count Mismatch

**Cause**

The value read from `SE` does not match the expected transaction segment count.

The validator compares:

```text
transaction.segments.length + 2
```

with:

```text
transaction.segmentCount
```

The extra two segments represent `ST` and `SE`.

For:

```text
ST
BIG
SE
```

the correct count is:

```text
3
```

---

#### Missing ISA Header

**Cause**

The parser did not detect an `ISA` interchange control header.

**Result**

Validation can report:

```text
Missing ISA interchange control header
```

---

### Validation Behavior

When validation is enabled, the current X12 validator checks:

- that an ISA interchange header was parsed;
- that the ISA control number contains exactly 9 characters;
- that each functional group's parsed transaction count matches the count read from `GE`;
- that each transaction's parsed business segment count plus `ST` and `SE` matches the count read from `SE`.

Validation problems are returned through the `validation` property rather than necessarily causing the node execution to fail.

> The current validator checks the control number length only; it does not verify that every character is numeric.


### Validation Messages

Possible validation findings include:

```text
Missing ISA interchange control header
```

```text
ISA control number must be 9 digits
```

```text
Group <index>: transaction count mismatch
```

```text
Group <groupIndex>, Transaction <transactionIndex>: segment count mismatch
```

### Error Messages

| Error | Description |
|-------|-------------|
| `X12 parsing failed: X12 input data is required` | No usable X12 document was provided. |
| `X12 parsing failed: <error message>` | The node wrapped an error raised while parsing the X12 document. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EDI Generate X12
- EDI Parse EDIFACT
- EDI Generate EDIFACT

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial release of the EDI Parse X12 documentation. |

<!-- /SECTION: changelog -->
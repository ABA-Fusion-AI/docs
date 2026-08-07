---
node_id: "edi-generate-edifact"
title: "EDI Generate EDIFACT"
description: "Generate UN/EDIFACT (ISO 9735) documents from structured JSON data."
category: "data-transformation-etl"
subcategory: "edi-structured-messages"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - edi
  - edifact
  - iso-9735
  - b2b
  - integration
  - structured-data
related_nodes:
  - edi-parse-edifact
  - edi-generate-x12
  - edi-parse-x12
---

<!-- SECTION: header -->
# EDI Generate EDIFACT

> **Type:** Action Node

Generate UN/EDIFACT documents from structured JSON data. The node converts interchange information, message definitions, segments, and data elements into an EDIFACT-formatted string.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EDI Generate EDIFACT** node generates an EDIFACT document from a structured JSON object.

Internally, the node uses `EDIFACTParser.generate()` to construct the EDIFACT interchange. The generated document contains a service string advice, interchange envelope, one or more messages, business segments, message trailers, and an interchange trailer.

### Key Features

- **Structured JSON Input:** Generates EDIFACT from structured interchange and message data.
- **Interchange Envelope:** Generates `UNB` and `UNZ` segments.
- **Message Envelope:** Generates `UNH` and `UNT` segments for each message.
- **Business Segments:** Supports custom EDIFACT segment tags and elements.
- **Composite Elements:** Converts arrays into colon-separated composite values.
- **Multiple Messages:** Supports multiple messages within one interchange.
- **Automatic Message Count:** Uses the number of items in `messages` when generating `UNZ`.
- **Plain Text Output:** Returns the complete generated EDIFACT document as a string.

### Generated Structure

The generator produces a document in this general order:

```text
UNA
UNB
UNH
Business segments
UNT
UNZ
```

When multiple messages are supplied:

```text
UNA
UNB
UNH
Business segments
UNT
UNH
Business segments
UNT
UNZ
```

### Separators Used by the Generator

The current implementation uses:

| Purpose | Character |
|---------|-----------|
| Component separator | `:` |
| Data element separator | `+` |
| Release character | `?` |
| Segment terminator | `'` |

These values are defined internally by the EDIFACT parser.

### Use Cases

- Generate EDIFACT invoices
- Create EDI documents from application data
- Build B2B integration workflows
- Generate logistics or supply-chain messages
- Prepare EDIFACT payloads for external systems
- Convert structured business data into EDIFACT syntax
- Integrate ERP or EDI processing systems

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `data` | `any` | ✅ Yes | — | Structured EDIFACT data. When not configured, the node attempts to use the incoming workflow object. |

Although the schema accepts `any`, the generator expects the value to follow the structure described below.

### Root Structure

The generator expects:

```json
{
  "interchange": {
    "syntaxIdentifier": ["UNOC", "3"],
    "sender": ["FUSIONAI"],
    "recipient": ["CLIENT01"],
    "dateTime": ["260807", "1400"],
    "controlReference": "000000001"
  },
  "messages": []
}
```

### Interchange Structure

The `interchange` object is used when generating the `UNB` and `UNZ` segments.

| Field | Expected Type | Used By | Description |
|-------|---------------|---------|-------------|
| `syntaxIdentifier` | `string \| array` | `UNB` | Syntax identifier composite. |
| `sender` | `string \| array` | `UNB` | Sender identification. |
| `recipient` | `string \| array` | `UNB` | Recipient identification. |
| `dateTime` | `string \| array` | `UNB` | Interchange date and time composite. |
| `controlReference` | `string` | `UNB`, `UNZ` | Interchange control reference. |

Example:

```json
{
  "syntaxIdentifier": ["UNOC", "3"],
  "sender": ["FUSIONAI"],
  "recipient": ["CLIENT01"],
  "dateTime": ["260807", "1400"],
  "controlReference": "000000001"
}
```

### Message Structure

The `messages` property must contain the messages to generate.

Each message is expected to contain:

| Field | Expected Type | Used By | Description |
|-------|---------------|---------|-------------|
| `messageReference` | `string` | `UNH`, `UNT` | Message reference number. |
| `messageType` | `string \| array` | `UNH` | Message type or composite message identifier. |
| `segmentCount` | `number` | `UNT` | Segment count written to the message trailer. |
| `segments` | `array` | Message body | Business segments generated between `UNH` and `UNT`. |

Example:

```json
{
  "messageReference": "1",
  "messageType": ["INVOIC", "D", "96A", "UN"],
  "segmentCount": 3,
  "segments": [
    {
      "tag": "BGM",
      "elements": [
        ["380"],
        "INV-1001",
        "9"
      ]
    }
  ]
}
```

### Segment Structure

Each object in `segments` contains:

| Field | Expected Type | Description |
|-------|---------------|-------------|
| `tag` | `string` | Segment identifier such as `BGM`, `DTM`, or `NAD`. |
| `elements` | `array` | Elements written after the segment tag. |

Example:

```json
{
  "tag": "BGM",
  "elements": [
    ["380"],
    "INV-1001",
    "9"
  ]
}
```

This generates a segment equivalent to:

```text
BGM+380+INV-1001+9'
```

### Composite Elements

Array values are joined using the component separator `:`.

For example:

```json
["INVOIC", "D", "96A", "UN"]
```

becomes:

```text
INVOIC:D:96A:UN
```

Similarly:

```json
["260807", "1400"]
```

becomes:

```text
260807:1400
```

### Segment Count

The generator writes `segmentCount` directly into the `UNT` segment.

For the tested message:

```text
UNH
BGM
UNT
```

the value is:

```json
"segmentCount": 3
```

The generation method itself does not recalculate or validate this value.

### Message Count

The `UNZ` message count is calculated automatically from:

```text
data.messages.length
```

You do not need to provide a separate interchange message count.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs


| Input | Type | Description |
|-------|------|-------------|
| `data` | `any` | Structured EDIFACT data configured in the node. |
| `input` | `object` | Incoming workflow data used when no configured `data` value is available. |

The node first uses the configured `data` value. If no configured value is available and the incoming workflow data is an object, the incoming object is used instead.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `result` | `string` | Generated EDIFACT document. |
| `format` | `string` | Output format identifier. Always `EDIFACT` on successful generation. |

### Successful Response

For a successful generation, the node returns:

```json
{
  "result": "UNA...UNB...UNH...UNT...UNZ...",
  "format": "EDIFACT"
}
```

The exact EDIFACT string depends on the supplied interchange, messages, segments, and elements.

### Tested Input

The following structure was used successfully with the node:

```json
{
  "interchange": {
    "syntaxIdentifier": [
      "UNOC",
      "3"
    ],
    "sender": [
      "FUSIONAI"
    ],
    "recipient": [
      "CLIENT01"
    ],
    "dateTime": [
      "260807",
      "1400"
    ],
    "controlReference": "000000001"
  },
  "messages": [
    {
      "messageReference": "1",
      "messageType": [
        "INVOIC",
        "D",
        "96A",
        "UN"
      ],
      "segmentCount": 3,
      "segments": [
        {
          "tag": "BGM",
          "elements": [
            [
              "380"
            ],
            "INV-1001",
            "9"
          ]
        }
      ]
    }
  ]
}
```

### Generated Sections

The tested input generates:

```text
UNA...
UNB+UNOC:3+FUSIONAI+CLIENT01+260807:1400+000000001'
UNH+1+INVOIC:D:96A:UN'
BGM+380+INV-1001+9'
UNT+3+1'
UNZ+1+000000001'
```

The exact `UNA` prefix should be treated as implementation-generated output rather than manually reconstructed.

### Error Response

Errors raised during generation are wrapped with:

```text
EDIFACT generation failed: <error message>
```

For example, an invalid object without the expected `interchange` structure can produce an error such as:

```text
EDIFACT generation failed: Cannot read properties of undefined (reading 'syntaxIdentifier')
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Generate an INVOIC Message

**Configuration**

```json
{
  "interchange": {
    "syntaxIdentifier": ["UNOC", "3"],
    "sender": ["FUSIONAI"],
    "recipient": ["CLIENT01"],
    "dateTime": ["260807", "1400"],
    "controlReference": "000000001"
  },
  "messages": [
    {
      "messageReference": "1",
      "messageType": ["INVOIC", "D", "96A", "UN"],
      "segmentCount": 3,
      "segments": [
        {
          "tag": "BGM",
          "elements": [
            ["380"],
            "INV-1001",
            "9"
          ]
        }
      ]
    }
  ]
}
```

The output contains:

```text
UNB+UNOC:3+FUSIONAI+CLIENT01+260807:1400+000000001'
UNH+1+INVOIC:D:96A:UN'
BGM+380+INV-1001+9'
UNT+3+1'
UNZ+1+000000001'
```

---

### Example: Add a Date Segment

A second business segment can be added:

```json
{
  "messageReference": "1",
  "messageType": ["INVOIC", "D", "96A", "UN"],
  "segmentCount": 4,
  "segments": [
    {
      "tag": "BGM",
      "elements": [
        ["380"],
        "INV-1001",
        "9"
      ]
    },
    {
      "tag": "DTM",
      "elements": [
        ["137", "20260807", "102"]
      ]
    }
  ]
}
```

The additional segment is generated as:

```text
DTM+137:20260807:102'
```

Because the message now contains:

```text
UNH
BGM
DTM
UNT
```

the example uses:

```json
"segmentCount": 4
```

---

### Example: Multiple Messages

The `messages` array may contain more than one message:

```json
{
  "interchange": {
    "syntaxIdentifier": ["UNOC", "3"],
    "sender": ["FUSIONAI"],
    "recipient": ["CLIENT01"],
    "dateTime": ["260807", "1400"],
    "controlReference": "000000002"
  },
  "messages": [
    {
      "messageReference": "1",
      "messageType": ["INVOIC", "D", "96A", "UN"],
      "segmentCount": 3,
      "segments": [
        {
          "tag": "BGM",
          "elements": [
            ["380"],
            "INV-1001",
            "9"
          ]
        }
      ]
    },
    {
      "messageReference": "2",
      "messageType": ["INVOIC", "D", "96A", "UN"],
      "segmentCount": 3,
      "segments": [
        {
          "tag": "BGM",
          "elements": [
            ["380"],
            "INV-1002",
            "9"
          ]
        }
      ]
    }
  ]
}
```

The generator writes:

```text
UNZ+2+000000002'
```

because two messages are present.

---

### Example: Composite Element

A segment element represented as:

```json
["137", "20260807", "102"]
```

is generated as:

```text
137:20260807:102
```

---

### Example: Scalar Element

A scalar value:

```json
"INV-1001"
```

is written directly as:

```text
INV-1001
```

---

### Example: Invalid Structure

A value such as:

```json
{
  "messageType": "INVOIC"
}
```

does not contain the `interchange` object expected by the generator.

Generation therefore fails and the node wraps the parser error:

```text
EDIFACT generation failed: <error message>
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Generate an EDIFACT Document
```

### Common Patterns

- **Generate EDIFACT:** Manual Trigger → EDI Generate EDIFACT → Log
- **Invoice Generation:** Manual Trigger → EDI Generate EDIFACT → Log
- **Multiple Message Generation:** Manual Trigger → EDI Generate EDIFACT → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "EDIFACT data structure is required"

**Cause**

No EDIFACT data structure was provided to the generator.

**Solution**

Provide a structured object containing at least:

```json
{
  "interchange": {},
  "messages": []
}
```

For a meaningful EDIFACT interchange, provide the expected interchange fields such as `syntaxIdentifier`, `sender`, `recipient`, `dateTime`, and `controlReference`.

---

#### "Cannot read properties of undefined (reading 'syntaxIdentifier')"

**Cause**

The generator attempted to read:

```text
data.interchange.syntaxIdentifier
```

but `interchange` was missing.

**Solution**

Provide an `interchange` object:

```json
{
  "interchange": {
    "syntaxIdentifier": ["UNOC", "3"],
    "sender": ["FUSIONAI"],
    "recipient": ["CLIENT01"],
    "dateTime": ["260807", "1400"],
    "controlReference": "000000001"
  },
  "messages": []
}
```

---

#### Incorrect Segment Count

**Cause**

The supplied `segmentCount` does not represent the intended number of segments in the message.

The generator writes this value directly and does not recalculate it.

**Solution**

Count:

- `UNH`
- every business segment
- `UNT`

For one business segment:

```text
UNH
BGM
UNT
```

use:

```json
"segmentCount": 3
```

For two business segments:

```text
UNH
BGM
DTM
UNT
```

use:

```json
"segmentCount": 4
```

---

#### Missing Message Reference

**Cause**

`messageReference` was omitted.

The generator uses the value in both `UNH` and `UNT`.

**Solution**

Provide a consistent value:

```json
"messageReference": "1"
```

---

#### Missing Control Reference

**Cause**

`controlReference` was omitted from the interchange.

The generator uses it in both the `UNB` and `UNZ` segments.

**Solution**

Provide:

```json
"controlReference": "000000001"
```

---

#### Unexpected Composite Formatting

**Cause**

An array was supplied where a scalar value was intended, or a scalar value was supplied where a composite was expected.

**Solution**

Use an array for colon-separated composite elements:

```json
["INVOIC", "D", "96A", "UN"]
```

which becomes:

```text
INVOIC:D:96A:UN
```

Use a scalar for a normal data element:

```json
"INV-1001"
```

---

### Error Messages

| Error | Description |
|-------|-------------|
| `EDIFACT generation failed: EDIFACT data structure is required` | No usable EDIFACT data was provided. |
| `EDIFACT generation failed: Cannot read properties of undefined (reading 'syntaxIdentifier')` | The expected `interchange` object is missing. |
| `EDIFACT generation failed: <error message>` | The node wrapped an error raised while generating the EDIFACT document. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EDI Parse EDIFACT
- EDI Generate X12
- EDI Parse X12

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial release of the EDI Generate EDIFACT documentation. |

<!-- /SECTION: changelog -->
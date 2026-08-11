---
node_id: "edi-parse-edifact"
title: "EDI Parse EDIFACT"
description: "Parse UN/EDIFACT (ISO 9735) documents with optional validation."
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
  - parsing
  - validation
  - structured-data
related_nodes:
  - edi-generate-edifact
  - edi-parse-x12
  - edi-generate-x12
---

<!-- SECTION: header -->
# EDI Parse EDIFACT

> **Category:** Data Transformation (ETL) | **Type:** Action Node

Parse UN/EDIFACT (ISO 9735) documents into structured data. The node supports UNA service string advice, interchange headers, messages, business segments, trailers, and optional validation.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EDI Parse EDIFACT** node converts a UN/EDIFACT document into structured workflow data.

Internally, the node uses `EDIFACTParser.parse()` to detect EDIFACT separators, parse the interchange envelope, extract messages and business segments, and optionally validate the parsed document.

### Key Features

- **EDIFACT Parsing:** Parses UN/EDIFACT documents into structured objects.
- **UNA Support:** Reads UNA service string advice when present.
- **Interchange Parsing:** Extracts information from `UNB` and `UNZ`.
- **Message Parsing:** Extracts `UNH` and `UNT` message information.
- **Business Segments:** Parses segments contained between `UNH` and `UNT`.
- **Composite Elements:** Splits composite values using the configured component separator.
- **Message Reference Check:** Verifies that `UNT` references the current `UNH` message.
- **Optional Validation:** Runs EDIFACT-specific validation when enabled.
- **Flexible Input Source:** Can read the EDIFACT document from node configuration or incoming workflow data.

### Parsed Structure

A successful parse returns data in this general structure:

```text
standard
version
interchange
messages
validation    ← when Validate is enabled
```

The interchange contains information extracted from `UNB` and `UNZ`, while each message contains its message reference, message type, business segments, and segment count.

### Default Separators

Without UNA, the parser uses:

| Purpose | Character |
|---------|-----------|
| Component separator | `:` |
| Data element separator | `+` |
| Release character | `?` |
| Segment terminator | `'` |

When the document starts with `UNA`, these separators are read from the UNA service string advice.

### Use Cases

- Parse EDIFACT invoices
- Read EDI messages from external systems
- Inspect interchange and message metadata
- Extract business segments
- Validate EDIFACT message structure
- Convert EDIFACT documents into workflow-friendly objects
- Process B2B or ERP integration messages

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputData` | `string` | ✅ Yes | — | UN/EDIFACT document to parse. |
| `validate` | `boolean` | ❌ No | `true` | Runs EDIFACT validation after parsing. |

### Input Data

Example:

```text
UNA:+.? 'UNB+UNOC:3+FUSIONAI+CLIENT01+260807:1400+000000001'UNH+1+INVOIC:D:96A:UN'BGM+380+INV-1001+9'UNT+3+1'UNZ+1+000000001'
```

### Input Source Behavior

The node first uses the configured `inputData`.

If no configured value is available, it can also use:

- an incoming workflow value when the input itself is a string;
- the `data` property when the incoming workflow input is an object.

For example, an incoming object may contain:

```json
{
  "data": "UNB+...'"
}
```

### Validation

When:

```text
Validate: true
```

the node:

1. parses the EDIFACT document;
2. runs `parser.validate(parsed)`;
3. appends the validation result to the output.

When:

```text
Validate: false
```

the node returns only the parsed EDIFACT structure.

### UNA Service String Advice

When the input starts with `UNA`, the parser reads the separators directly from the document.

The parser reads:

```text
UNA
│
├── component separator
├── data element separator
├── release character
└── segment terminator
```

After processing the UNA prefix, parsing continues with the remaining segments.

### Current Line-Break Behavior

With the current parser implementation, EDIFACT input should be provided as a continuous string without line breaks between segments.

Recommended:

```text
UNB+...'UNH+...'BGM+...'UNT+...'UNZ+...'
```

Avoid:

```text
UNB+...'
UNH+...'
BGM+...'
UNT+...'
UNZ+...'
```

The parser does not trim the parsed segment tag before matching values such as `UNB`, `UNH`, `UNT`, and `UNZ`. Leading line-break characters can therefore prevent those segments from being recognized.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `inputData` | `string` | EDIFACT document configured directly in the node. |
| `input` | `string \| object` | Incoming workflow data used when `inputData` is not configured. |

If the incoming value is an object, the node reads its `data` property.

### Outputs

The parsed output contains:

| Field | Type | Description |
|-------|------|-------------|
| `standard` | `string` | Parsed format identifier. Returns `EDIFACT`. |
| `version` | `string` | EDIFACT syntax version extracted from the UNB syntax identifier. |
| `interchange` | `object` | Parsed interchange metadata. |
| `messages` | `array` | Parsed EDIFACT messages. |
| `validation` | `object` | Validation result when `validate` is enabled. |

### Interchange Structure

The `interchange` object can contain:

| Field | Description |
|-------|-------------|
| `syntaxIdentifier` | Parsed syntax identifier. |
| `sender` | Sender identification. |
| `recipient` | Recipient identification. |
| `dateTime` | Interchange date and time. |
| `controlReference` | Interchange control reference. |
| `recipientReference` | Optional recipient reference. |
| `applicationReference` | Optional application reference. |
| `priorityCode` | Optional priority code. |
| `acknowledgementRequest` | Optional acknowledgment request value. |
| `testIndicator` | Optional test indicator. |
| `messageCount` | Message count read from `UNZ`. |

### Message Structure

Each item in `messages` contains:

| Field | Type | Description |
|-------|------|-------------|
| `messageReference` | `string` | Reference extracted from `UNH`. |
| `messageType` | `string \| array` | Message type or composite message identifier. |
| `segments` | `array` | Business segments found inside the message. |
| `segmentCount` | `number` | Segment count read from `UNT`. |

### Segment Structure

Each parsed segment contains:

| Field | Type | Description |
|-------|------|-------------|
| `tag` | `string` | EDIFACT segment identifier. |
| `elements` | `array` | Parsed segment elements. |

Composite values are returned as arrays.

For example:

```text
INVOIC:D:96A:UN
```

is parsed into:

```json
[
  "INVOIC",
  "D",
  "96A",
  "UN"
]
```

### Successful Response

For the tested document, the parsed structure contains values equivalent to:

```json
{
  "standard": "EDIFACT",
  "version": "3",
  "interchange": {
    "syntaxIdentifier": [
      "UNOC",
      "3"
    ],
    "sender": "FUSIONAI",
    "recipient": "CLIENT01",
    "dateTime": [
      "260807",
      "1400"
    ],
    "controlReference": "000000001",
    "messageCount": 1
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
      "segments": [
        {
          "tag": "BGM",
          "elements": [
            "380",
            "INV-1001",
            "9"
          ]
        }
      ],
      "segmentCount": 3
    }
  ]
}
```

When validation is enabled, the result additionally contains:

```json
{
  "validation": {
    "valid": true
  }
}
```

Additional validation information may also be included by the parser.

### Error Response

Errors are wrapped by the node using:

```text
EDIFACT parsing failed: <error message>
```

If no usable input is available:

```text
EDIFACT parsing failed: EDIFACT input data is required
```

A mismatched `UNT` message reference can produce:

```text
EDIFACT parsing failed: Message reference mismatch: expected <expected>, got <actual>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Parse an EDIFACT Invoice

**Input Data**

```text
UNA:+.? 'UNB+UNOC:3+FUSIONAI+CLIENT01+260807:1400+000000001'UNH+1+INVOIC:D:96A:UN'BGM+380+INV-1001+9'UNT+3+1'UNZ+1+000000001'
```

**Configuration**

```text
Validate: true
```

The node parses the interchange and returns one `INVOIC` message containing one `BGM` business segment.

---

### Example: Parse Without Validation

**Configuration**

```text
Validate: false
```

The node returns the parsed EDIFACT structure without appending a `validation` property.

---

### Example: Composite Message Type

Input:

```text
UNH+1+INVOIC:D:96A:UN'
```

The `messageType` becomes:

```json
[
  "INVOIC",
  "D",
  "96A",
  "UN"
]
```

---

### Example: Composite Date and Time

Input:

```text
UNB+UNOC:3+FUSIONAI+CLIENT01+260807:1400+000000001'
```

The parsed `dateTime` becomes:

```json
[
  "260807",
  "1400"
]
```

---

### Example: Incoming String

When `inputData` is not configured, an incoming string can be parsed directly:

```text
UNA:+.? 'UNB+...'
```

---

### Example: Incoming Object

When `inputData` is not configured, the node can read:

```json
{
  "data": "UNA:+.? 'UNB+...'"
}
```

---

### Example: Invalid Message Reference

Input containing:

```text
UNH+1+INVOIC:D:96A:UN'
...
UNT+3+2'
```

causes an error because the `UNT` reference does not match the `UNH` reference.

The node returns an error equivalent to:

```text
EDIFACT parsing failed: Message reference mismatch: expected 1, got 2
```

---

### Example: Multiline Input

With the current implementation, this form should be avoided:

```text
UNB+...'
UNH+...'
BGM+...'
UNT+...'
```

Prefer:

```text
UNB+...'UNH+...'BGM+...'UNT+...'
```

This prevents leading line-break characters from becoming part of the parsed segment tag.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Parse and Validate an EDIFACT Document
```

### Common Patterns

- **Parse EDIFACT:** Manual Trigger → EDI Parse EDIFACT → Log
- **Validate EDIFACT:** Manual Trigger → EDI Parse EDIFACT → Log
- **Inspect EDIFACT Messages:** Manual Trigger → EDI Parse EDIFACT → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "EDIFACT input data is required"

**Cause**

No configured `inputData`, incoming string, or incoming object containing `data` was available.

**Solution**

Provide a valid EDIFACT document through the node configuration or workflow input.

---

#### "No messages found"

**Cause**

Validation did not find any parsed `UNH` messages.

One possible cause with the current parser is using line breaks between EDIFACT segments.

For example:

```text
UNB+...'
UNH+...'
```

can cause the second segment tag to contain a leading line-break character.

**Solution**

Provide the document on one line:

```text
UNB+...'UNH+...'
```

Also verify that the document actually contains `UNH` and `UNT` segments.

---

#### Message Reference Mismatch

**Cause**

The reference in `UNT` differs from the current `UNH` message reference.

Example:

```text
UNH+1+INVOIC:D:96A:UN'
...
UNT+3+2'
```

**Solution**

Use the same reference in `UNH` and `UNT`.

Correct:

```text
UNH+1+INVOIC:D:96A:UN'
...
UNT+3+1'
```

---

#### Segment Count Mismatch

**Cause**

When validation is enabled, the parser compares:

```text
message.segments.length + 2
```

with:

```text
message.segmentCount
```

The additional two segments represent `UNH` and `UNT`.

For one business segment:

```text
UNH
BGM
UNT
```

the correct count is:

```text
3
```

For two business segments:

```text
UNH
BGM
DTM
UNT
```

the correct count is:

```text
4
```

---

#### Version Is Empty

**Cause**

The parser extracts the version from the second component of the `UNB` syntax identifier.

For example:

```text
UNOC:3
```

produces:

```text
version: 3
```

If the syntax identifier does not contain a second component, the version can remain empty.

---

#### Missing or Unrecognized UNB Header

**Cause**

The EDIFACT document does not contain a recognizable `UNB` segment, or the segment was not parsed correctly.

**Solution**

Verify that the interchange contains a valid `UNB` segment after any optional `UNA` service string advice.

> With the current parser implementation, the validation result may not explicitly report a missing `UNB` header because the interchange object is initialized before segment parsing.

---

### Validation Behavior

When validation is enabled, the EDIFACT-specific validator currently checks:

- that messages are present;
- that each message's segment count is consistent.

The validator also contains a check intended to detect a missing interchange header, but the current parser initializes the interchange object before parsing, so that condition may not detect a missing `UNB` segment.

Validation problems are returned through the validation result. Parsing errors such as message-reference mismatches are thrown and wrapped by the node.

### Error Messages

| Error | Description |
|-------|-------------|
| `EDIFACT parsing failed: EDIFACT input data is required` | No EDIFACT input was available. |
| `EDIFACT parsing failed: Message reference mismatch: expected <expected>, got <actual>` | `UNT` does not reference the current `UNH` message. |
| `EDIFACT parsing failed: <error message>` | The node wrapped an error raised while parsing the EDIFACT document. |

### Validation Messages

Validation may report messages such as:

```text
No messages found
```

or:

```text
Message <index>: segment count mismatch
```

These validation findings do not necessarily throw an execution error; they are returned through the validation result.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- EDI Generate EDIFACT
- EDI Parse X12
- EDI Generate X12

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial release of the EDI Parse EDIFACT documentation. |

<!-- /SECTION: changelog -->
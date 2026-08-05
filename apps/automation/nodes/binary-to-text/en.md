---
node_id: "binary-to-text"
title: "Binary to Text"
description: "Convert binary data into text using a selected character encoding"
category: "data-transformation"
subcategory: "binary"
version: "1.0.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - binary
  - text
  - encoding
  - transformation
related_nodes:
  - text-to-binary
  - function
  - webhook
---

<!-- SECTION: header -->

# Binary to Text

> **Category:** Data Transformation | **Type:** Action Node

Convert binary input into readable text using a selected encoding.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `data` | `binary \| buffer \| object` | ✅ Yes | — | Binary data to convert |
| `encoding` | `enum` | ❌ No | `utf8` | Character encoding used to decode the binary value |

### Supported Encodings

Common values: `utf8`, `ascii`, `utf16le`, `latin1`, `base64`, and `hex`.

The selected encoding must match the incoming data format.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs and Outputs

### Input

| Port | Description |
| --- | --- |
| `input` | Receives binary data or an object containing binary content |

Current expression:

```json
{
  "$expr": "output",
  "node": "Function",
  "outputId": "success"
}
```

### Success Output

```json
{
  "text": "Decoded text content",
  "encoding": "utf8",
  "length": 20
}
```

### Error Output

```json
{
  "text": null,
  "errorMessage": "Unable to convert binary data",
  "errorCode": "BINARY_TO_TEXT_ERROR"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: example -->

## Example

### Configuration

```json
{
  "data": {
    "$expr": "output",
    "node": "Function",
    "outputId": "success"
  },
  "encoding": "utf8"
}
```

### Expected Output

```json
{
  "text": "Hello",
  "encoding": "utf8",
  "length": 5
}
```

<!-- /SECTION: example -->

---

<!-- SECTION: errors -->

## Error Handling

| Error | Code |
| --- | --- |
| Missing input data | `MISSING_BINARY_DATA` |
| Unsupported encoding | `INVALID_ENCODING` |
| Invalid binary value | `INVALID_BINARY_DATA` |
| Decoding failure | `BINARY_TO_TEXT_ERROR` |

Successful conversions go to `success`; invalid input or decoding failures go to `error`.

<!-- /SECTION: errors -->

---

<!-- SECTION: notes -->

## Notes

- Use `utf8` for normal text and JSON payloads.
- Use `base64` only when the input contains Base64-encoded content.
- The node only decodes binary data; it does not perform OCR or document parsing.
- Apply file-size limits for large payloads.

<!-- /SECTION: notes -->

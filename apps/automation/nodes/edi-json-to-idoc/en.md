---
node_id: "json-to-idoc"
title: "JSON to IDoc Converter"
description: "Converts JSON data to SAP IDoc XML format. Supports DESADV, SHPMNT, ZFREINV, ZLEITRACK, STPPOD, STATUS, and ZINVRSP message types."
category: "edi"
subcategory: "SAP Integration"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - sap
  - idoc
  - edi
  - xml
  - logistics
  - integration
related_nodes:
  - idoc
  - edi-parse-edifact
  - http-request
  - function
---

<!-- SECTION: overview -->
# JSON to IDoc Converter

> **Category:** EDI &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Convert structured JSON data into **SAP IDoc XML format** for direct integration with SAP systems. Supports six standard and custom IDoc message types used in logistics, shipping, invoicing, and tracking workflows.

### Use Cases

- **Logistics Integration:** Convert shipment or delivery data from internal systems into DESADV or SHPMNT IDocs for SAP.
- **Invoice Processing:** Generate ZFREINV or ZINVRSP IDoc XML from billing records and send them to an SAP ERP instance.
- **Track & Trace:** Transform shipment tracking events into ZLEITRACK IDocs for SAP Transportation Management.
- **Status Updates:** Push delivery or order status changes to SAP using STATUS IDocs from any upstream data source.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `inputData` | `object` | No | — | The JSON data to convert. If omitted, the node reads from the incoming workflow input. |
| `idocType` | `enum` | Yes | `DESADV` | The SAP IDoc message type to generate. See supported types below. |
| `environment` | `enum` | No | `test` | Target environment: `test` or `prod`. Affects envelope metadata in the generated XML. |
| `senderPOR` | `string` | No | — | Sender Port (POR) identifier. Used in the IDoc control record. |
| `receiverPOR` | `string` | No | — | Receiver Port (POR) identifier. Used in the IDoc control record. |
| `senderPRN` | `string` | No | — | Sender Partner Number. Identifies the sending system in the IDoc envelope. |
| `receiverPRN` | `string` | No | — | Receiver Partner Number. Identifies the receiving SAP system. |
| `prettyPrint` | `boolean` | No | `true` | Format the output XML with indentation for readability. |
| `includeEmptyFields` | `boolean` | No | `false` | Include XML elements for fields that have no value. |

### Supported IDoc Types

| IDoc Type | Description |
|-----------|-------------|
| `DESADV` | Despatch Advice — used for advance shipping notifications (ASN). |
| `SHPMNT` | Shipment — used for shipment data transfer between logistics systems. |
| `ZFREINV` | Custom freight invoice message type. |
| `ZLEITRACK` | Custom shipment tracking/tracing message type. |
| `STPPOD` | Stop Proof of Delivery — used to confirm delivery at a stop. |
| `STATUS` | Status message — used to communicate delivery or order status updates. |
| `ZINVRSP` | Custom invoice response message type. |

### Environment

| Value | Description |
|-------|-------------|
| `test` | Generates IDoc XML with test environment identifiers. Use for development and QA. |
| `prod` | Generates IDoc XML with production identifiers. Use for live SAP integration. |

### Output Formatting

- **`prettyPrint: true`** — Outputs human-readable XML with proper indentation. Recommended for debugging.
- **`prettyPrint: false`** — Outputs compact single-line XML. Recommended for production transport to reduce payload size.
- **`includeEmptyFields: false`** — Omits XML elements with no value (default, cleaner output).
- **`includeEmptyFields: true`** — Includes all fields even if empty. Useful when the receiving SAP system expects fixed structure.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | The JSON payload to convert. Used if `inputData` is not explicitly set. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `string` | The generated SAP IDoc XML document as a string. |
| `error` | `Error` | Emitted if the input data is invalid, a required field is missing, or the conversion fails. |

### Output Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<IDOC BEGIN="1">
  <EDI_DC40 SEGMENT="1">
    <TABNAM>EDI_DC40</TABNAM>
    <MANDT>100</MANDT>
    <DOCNUM>0000000000000001</DOCNUM>
    <DOCREL>756</DOCREL>
    <STATUS>30</STATUS>
    <DIRECT>1</DIRECT>
    <OUTMOD>2</OUTMOD>
    <IDOCTYP>DESADV01</IDOCTYP>
    <MESTYP>DESADV</MESTYP>
    <SNDPOR>SENDER_PORT</SNDPOR>
    <SNDPRN>SENDER_PRN</SNDPRN>
    <RCVPOR>RECEIVER_PORT</RCVPOR>
    <RCVPRN>RECEIVER_PRN</RCVPRN>
  </EDI_DC40>
  <E1EDKA1 SEGMENT="1">
    <PARVW>LF</PARVW>
    <PARTN>1234567890</PARTN>
  </E1EDKA1>
</IDOC>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Convert JSON to SAP IDoc XML
```

### How it flows

1. **Manual Trigger** (or upstream node): Provides the JSON data to convert.
2. **JSON to IDoc Converter:** Receives the JSON, applies the selected `idocType` and routing parameters, and outputs the IDoc XML string.
3. **HTTP Request Node** (downstream): Posts the generated XML to an SAP PI/PO endpoint or a middleware system for further processing.

### Common Patterns

- **ERP Integration Pipeline:** Use a Function node to shape your internal data into the expected JSON structure, then convert it to IDoc XML and POST it to your SAP system via HTTP Request.
- **Environment Switching:** Use a workflow variable to dynamically set `environment` to `test` or `prod` based on a deployment flag, avoiding separate workflows for each environment.
- **Batch Processing:** Feed multiple records through a For Each node, convert each one to IDoc, and send them sequentially to SAP.
- **Monitoring & Logging:** After the conversion, log the XML output with a Log node before sending it to validate the generated structure.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Empty or malformed XML output
- **Cause:** The `inputData` JSON does not match the expected structure for the selected `idocType`.
- **Solution:** Enable `prettyPrint` and review the generated XML. Ensure the input JSON contains the required fields for the target IDoc type. Test with `environment: test` first.

#### Missing partner routing in the IDoc envelope
- **Cause:** `senderPOR`, `receiverPOR`, `senderPRN`, or `receiverPRN` are not set.
- **Solution:** Provide the partner numbers and port identifiers from your SAP partner profile configuration.

#### SAP rejects the IDoc on ingestion
- **Cause:** The `environment` is set to `test` but the XML was sent to the production SAP system, or vice versa.
- **Solution:** Ensure `environment` matches the target SAP system. Also verify that the `idocType` version is compatible with the configured SAP IDoc type definition.

#### Node receives no data
- **Cause:** Both `inputData` and the upstream input are empty.
- **Solution:** Connect an upstream node that provides the JSON payload, or explicitly set the `inputData` parameter via expression.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [IDoc](./idoc.md) – Parse or handle incoming SAP IDoc documents
- [EDI Parse EDIFACT](./edi-parse-edifact.md) – Parse EDIFACT EDI messages
- [HTTP Request](./http-request.md) – Send the generated IDoc XML to an SAP PI/PO or middleware endpoint
- [Function](./function.md) – Shape and transform JSON data before passing it to this node

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial documentation |

<!-- /SECTION: changelog -->

---
node_id: "banking-validator"
title: "Banking Validator"
description: "Validate IBAN and SWIFT/BIC codes with format checking and MOD-97 checksum verification."
category: "peer-only"
subcategory: "business-utils-nodes"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:
  - banking
  - validation
  - iban
  - swift
  - bic
  - format
  - Peer-only
  - action
related_nodes:
  - function
  - webhook-trigger
  - http-request
---

<!-- SECTION: overview -->
# Banking Validator

> **Category:** Peer-only  | **Type:** Action Node

Validate IBAN (International Bank Account Number) and SWIFT/BIC (Bank Identifier Code) values with structural format checking and ISO 7064 Modulo 97 checksum calculations.

The Banking Validator node ensures bank account details submitted in automation workflows are valid, well-formatted, and compliant with international banking standards before triggering downstream payouts, invoices, or ERP integrations.

### Use Cases

- Validate customer or supplier IBANs during onboarding or payment processing
- Verify SWIFT/BIC codes before initiating international wire transfers
- Format raw IBAN input strings into standardized 4-character block representations for display or documents
- Prevent failed banking transactions by checking MOD-97 checksums upfront
- Clean and normalize banking identifiers in automated data pipelines

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `validateIBAN` | Operation to execute (`validateIBAN`, `validateSWIFT`, `formatIBAN`) |
| `iban` | `string` | ❌ No | `""` | IBAN string to validate or format (Required for `validateIBAN` and `formatIBAN`) |
| `swift` | `string` | ❌ No | `""` | SWIFT/BIC code to validate (Required for `validateSWIFT`) |

### Supported Operations

| Operation | Description |
|-----------|-------------|
| `validateIBAN` | Validates IBAN length, country code structure, and ISO 7064 MOD-97 checksum |
| `validateSWIFT` | Validates SWIFT/BIC code format (8 or 11 alphanumeric characters) |
| `formatIBAN` | Formats a raw IBAN into 4-character blocks separated by spaces |

---

### Operation: Validate IBAN

Validates an International Bank Account Number (IBAN) using length rules, country structure matching, and the ISO 7064 MOD-97 algorithm.

#### Parameters

| Field | Required | Description |
|-------|----------|-------------|
| `iban` | ✅ Yes | IBAN string to validate |

#### Output (Valid IBAN)

```json
{
  "success": true,
  "valid": true,
  "iban": "DE89370400440532013000",
  "formatted": "DE89 3704 0044 0532 0130 00",
  "country": "DE",
  "checksumValid": true,
  "structureValid": true,
  "lengthValid": true,
  "length": 22
}
```

#### Output (Invalid IBAN - Checksum Failure)

```json
{
  "success": true,
  "valid": false,
  "iban": "DE00370400440532013000",
  "country": "DE",
  "checksumValid": false,
  "structureValid": true,
  "lengthValid": true,
  "error": "Invalid MOD-97 checksum"
}
```

> **Note**
>
> The IBAN validator automatically trims leading/trailing whitespace and converts lowercase letters to uppercase before performing validation.

---

### Operation: Validate SWIFT / BIC

Validates SWIFT / BIC codes according to ISO 9362 rules.

#### Parameters

| Field | Required | Description |
|-------|----------|-------------|
| `swift` | ✅ Yes | SWIFT/BIC code string to validate |

#### Output (Valid SWIFT Code)

```json
{
  "success": true,
  "valid": true,
  "swift": "BMCEMAMC",
  "bankCode": "BMCE",
  "countryCode": "MA",
  "locationCode": "MC",
  "branchCode": "XXX"
}
```

#### Output (Invalid SWIFT Code)

```json
{
  "success": true,
  "valid": false,
  "swift": "INVALID123",
  "error": "SWIFT code must be 8 or 11 alphanumeric characters (4 bank letters, 2 country letters, 2 location code, optional 3 branch code)"
}
```

---

### Operation: Format IBAN

Formats a valid or raw IBAN string into a human-readable format separated into 4-character blocks.

#### Parameters

| Field | Required | Description |
|-------|----------|-------------|
| `iban` | ✅ Yes | IBAN string to format |

#### Output

```json
{
  "success": true,
  "valid": true,
  "formatted": "DE89 3704 0044 0532 0130 00",
  "compact": "DE89370400440532013000",
  "country": "DE"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Workflow data used to populate IBAN or SWIFT fields dynamically via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when validation or formatting completes |
| `error` | `Error` | Emitted if an unhandled node execution error occurs |

### Output Summary

| Operation | Output Key | Description |
|-----------|------------|-------------|
| `validateIBAN` | `valid`, `iban`, `formatted`, `country`, `checksumValid` | Validation state & extracted components |
| `validateSWIFT` | `valid`, `swift`, `bankCode`, `countryCode`, `locationCode`, `branchCode` | SWIFT verification state & parsed segments |
| `formatIBAN` | `formatted`, `compact`, `country` | Formatted IBAN string representations |

---

### Example: Validate IBAN

**Configuration**

```json
{
  "operation": "validateIBAN",
  "iban": "MA43 0115 1900 0001 2050 0053 49"
}
```

**Output**

```json
{
  "success": true,
  "valid": true,
  "iban": "MA430115190000012050005349",
  "formatted": "MA43 0115 1900 0001 2050 0053 49",
  "country": "MA",
  "checksumValid": true,
  "structureValid": true,
  "lengthValid": true,
  "length": 28
}
```

---

### Example: Validate SWIFT / BIC

**Configuration**

```json
{
  "operation": "validateSWIFT",
  "swift": "BMCEMAMC"
}
```

**Output**

```json
{
  "success": true,
  "valid": true,
  "swift": "BMCEMAMC",
  "bankCode": "BMCE",
  "countryCode": "MA",
  "locationCode": "MC",
  "branchCode": "XXX"
}
```

---

### Example: Format IBAN

**Configuration**

```json
{
  "operation": "formatIBAN",
  "iban": "DE89370400440532013000"
}
```

**Output**

```json
{
  "success": true,
  "valid": true,
  "formatted": "DE89 3704 0044 0532 0130 00",
  "compact": "DE89370400440532013000",
  "country": "DE"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Validate Banking Details Before Payout

```json
{
  "nodes": [
    {
      "id": "webhook-trigger",
      "type": "webhook-trigger"
    },
    {
      "id": "validate-iban",
      "type": "banking-validator",
      "config": {
        "operation": "validateIBAN",
        "iban": "{{input.body.iban}}"
      }
    },
    {
      "id": "validate-swift",
      "type": "banking-validator",
      "config": {
        "operation": "validateSWIFT",
        "swift": "{{input.body.swift}}"
      }
    },
    {
      "id": "process-payout",
      "type": "http-request"
    }
  ]
}
```

### Common Patterns

- Customer Payout Request → Validate IBAN → Validate SWIFT → Execute Transfer
- Supplier Onboarding → Validate IBAN & SWIFT → Save to Database
- Invoice Processing → Extract Banking Details → Format IBAN → Generate Receipt PDF
- Batch Payment CSV Processing → IBAN MOD-97 Filter → Route Valid Accounts to Gateway

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### IBAN string is required

**Cause**

The `validateIBAN` or `formatIBAN` operation was called with an empty or missing `iban` parameter.

**Solution**

Provide a valid IBAN string or bind it using expressions (e.g. `{{input.body.iban}}`).

---

### SWIFT code is required

**Cause**

The `validateSWIFT` operation was selected without supplying a `swift` parameter.

**Solution**

Provide an 8 or 11-character SWIFT/BIC code string.

---

### Invalid MOD-97 checksum

**Cause**

The IBAN checksum validation failed. Transposed digits, typos, or incorrect country codes break the ISO 7064 MOD-97 algorithm.

**Solution**

- Verify the IBAN digits against official bank documentation.
- Ensure the 2-letter country code at the start of the IBAN is correct.
- Check for transposed numbers or missing digits.

---

### Invalid SWIFT / BIC format

**Cause**

The provided SWIFT code does not match ISO 9362 structural requirements (4 bank letters + 2 country letters + 2 location characters + optional 3 branch characters).

**Solution**

- Ensure the SWIFT code has exactly 8 or 11 characters.
- Ensure the first 4 characters are letters representing the bank code.
- Ensure characters 5-6 represent a valid 2-letter ISO country code.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Execute custom JavaScript logic
- [Webhook Trigger](./webhook-trigger.md) – Receive incoming payment webhooks
- [HTTP Request](./http-request.md) – Connect to payment processing gateways

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-11 | Initial release |

<!-- /SECTION: changelog -->

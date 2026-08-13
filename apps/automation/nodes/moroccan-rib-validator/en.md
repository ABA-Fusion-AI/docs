---
node_id: "moroccan-rib-validator"
title: "Moroccan RIB Validator"
description: "Validate Moroccan RIB (Relevé d'Identité Bancaire) with bank detection."
category: "peer-only"
subcategory: "business-utils-nodes"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - moroccan-rib-validator
  - rib
  - morocco
  - banking
  - validation
  - bank-code
  - releve-identite-bancaire
  - peer-only
  - action
related_nodes:
  - banking-validator
  - function
  - if
  - http-request
---

<!-- SECTION: overview -->
# Moroccan RIB Validator

> **Category:** Peer-only &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Validate Moroccan bank account RIB (**Relevé d'Identité Bancaire**) numbers, verify Modulo 97 control key checksums, extract bank branch details, format RIB strings, and look up Moroccan financial institution details.

The **Moroccan RIB Validator** node enables workflows to validate 24-digit Moroccan RIB strings submitted in forms, invoices, supplier registration portals, or payroll pipelines. In Morocco, a standard RIB consists of a **3-digit Bank Code**, a **3-digit Counter/City Code**, a **16-digit Account Number**, and a **2-digit Control Key (Clé RIB)** calculated via Modulo 97.

### Key Capabilities

- **Validate RIB (`validateRIB`):** Verify 24-digit RIB length, numeric format, Modulo 97 checksum key validity, and automatically detect the issuing Moroccan bank (e.g., Attijariwafa Bank, Banque Populaire, BMCE / Bank of Africa, CIH, BMCI, Crédit Agricole, etc.).
- **Get Bank Information (`getBankInfo`):** Look up bank details (bank name, SWIFT code, country) by passing a 3-digit Moroccan bank code.
- **Get All Banks (`getAllBanks`):** List all recognized Moroccan banking institutions with their 3-digit bank codes and SWIFT identifiers.
- **Format RIB (`formatRIB`):** Standardize a 24-digit numeric string into formatted space-separated blocks (`007 780 1234567890123456 73`).

### Use Cases

- **Supplier & Merchant Onboarding:** Validate Moroccan bank account details automatically before storing them in CRM or ERP databases.
- **Payroll & Payout Processing:** Ensure employee or contractor RIB numbers are valid and check Modulo 97 checksums upfront to avoid returned transfers.
- **Invoice & Expense Audit:** Extract bank codes and counter codes from uploaded Moroccan invoices to verify vendor banking credentials.
- **Form Input Sanitization:** Format raw 24-digit user input into standardized 4-part space-delimited RIB strings for displays and reports.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `"validateRIB"` | The RIB operation to execute (see operations table below). |
| `rib` | `string` | ❌ No | — | The 24-digit Moroccan RIB string to validate or format. Required for `validateRIB` and `formatRIB`. Supports expressions. |
| `bankCode` | `string` | ❌ No | — | The 3-digit Moroccan bank code (e.g., `"007"`). Required for `getBankInfo`. Supports expressions. |

### Operations (`operation`)

| Operation | Value | Description |
|-----------|-------|-------------|
| **Validate RIB** | `validateRIB` | Validates a 24-digit Moroccan RIB, checks its Modulo 97 control key, and detects the issuing bank. |
| **Get Bank Info** | `getBankInfo` | Retrieves bank details (name, SWIFT code) for a specified 3-digit bank code. |
| **Get All Banks** | `getAllBanks` | Returns a list of all supported Moroccan banking institutions and their 3-digit codes. |
| **Format RIB** | `formatRIB` | Formats a raw 24-digit RIB string into standard space-separated groups (`3 3 16 2`). |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `string` | Workflow input data or trigger payload containing the target RIB number or bank code. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned when validation, formatting, or bank lookup completes successfully. |
| `error` | `object` | Returned if the request parameters are missing or structurally invalid. |

### Output Data Fields (`validateRIB`)

| Field | Type | Description |
|-------|------|-------------|
| `valid` | `boolean` | `true` if the RIB length, digit format, and Modulo 97 checksum key are all valid. |
| `rib` | `string` | Cleaned 24-digit input RIB string. |
| `formatted` | `string` | Standardized space-separated format (`007 780 1234567890123456 73`). |
| `bankCode` | `string` | Extracted 3-digit bank code (e.g., `"007"`). |
| `counterCode` | `string` | Extracted 3-digit counter/city code (e.g., `"780"`). |
| `accountNumber` | `string` | Extracted 16-digit account number. |
| `key` | `string` | Extracted 2-digit control key (Clé RIB). |
| `bank` | `object` | Detected bank details including `code`, `name`, and `swift`. |
| `checksumValid` | `boolean` | `true` if the Modulo 97 key matches the calculated control digits. |

---

### Configuration & Output Examples

#### Example 1: Validate a Valid Moroccan RIB (`validateRIB`)

**Configuration**
```json
{
  "operation": "validateRIB",
  "rib": "007780123456789012345673"
}
```

**Output (`success`)**
```json
{
  "valid": true,
  "rib": "007780123456789012345673",
  "formatted": "007 780 1234567890123456 73",
  "bankCode": "007",
  "counterCode": "780",
  "accountNumber": "1234567890123456",
  "key": "73",
  "bank": {
    "code": "007",
    "name": "Attijariwafa Bank",
    "swift": "BCMA"
  },
  "checksumValid": true
}
```

---

#### Example 2: Validate an Invalid RIB (Checksum Failure)

**Configuration**
```json
{
  "operation": "validateRIB",
  "rib": "007780123456789012345672"
}
```

**Output (`success`)**
```json
{
  "valid": false,
  "rib": "007780123456789012345672",
  "bankCode": "007",
  "counterCode": "780",
  "accountNumber": "1234567890123456",
  "key": "72",
  "bank": {
    "code": "007",
    "name": "Attijariwafa Bank",
    "swift": "BCMA"
  },
  "checksumValid": false,
  "error": "Invalid RIB checksum key"
}
```

---

#### Example 3: Get Bank Information (`getBankInfo`)

**Configuration**
```json
{
  "operation": "getBankInfo",
  "bankCode": "007"
}
```

**Output (`success`)**
```json
{
  "found": true,
  "bank": {
    "code": "007",
    "name": "Attijariwafa Bank",
    "swift": "BCMA",
    "country": "MA"
  }
}
```

---

#### Example 4: Get All Moroccan Banks (`getAllBanks`)

**Configuration**
```json
{
  "operation": "getAllBanks"
}
```

**Output (`success`)**
```json
[
  {
    "code": "007",
    "name": "Attijariwafa Bank",
    "swift": "BCMA"
  },
  {
    "code": "181",
    "name": "Banque Populaire",
    "swift": "BCCN"
  },
  {
    "code": "011",
    "name": "BMCE Bank / Bank of Africa",
    "swift": "BMCE"
  },
  {
    "code": "013",
    "name": "BMCI",
    "swift": "BMCI"
  },
  {
    "code": "019",
    "name": "CIH Bank",
    "swift": "CIH"
  },
  {
    "code": "021",
    "name": "Crédit Agricole du Maroc",
    "swift": "AGRI"
  },
  {
    "code": "022",
    "name": "Crédit du Maroc",
    "swift": "CDM"
  },
  {
    "code": "023",
    "name": "Société Générale Maroc",
    "swift": "SGE"
  },
  {
    "code": "350",
    "name": "Al Barid Bank",
    "swift": "PMA"
  }
]
```

---

#### Example 5: Format RIB String (`formatRIB`)

**Configuration**
```json
{
  "operation": "formatRIB",
  "rib": "007780123456789012345673"
}
```

**Output (`success`)**
```json
{
  "formatted": "007 780 1234567890123456 73",
  "compact": "007780123456789012345673",
  "bankCode": "007",
  "counterCode": "780",
  "accountNumber": "1234567890123456",
  "key": "73"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate Moroccan RIB and Retrieve Bank Details
```

### Sample Workflow: Manual Trigger → Moroccan RIB Validator → Log Result

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger"
    },
    {
      "id": "rib-validator",
      "type": "moroccan-rib-validator",
      "config": {
        "operation": "validateRIB",
        "rib": "007780123456789012345673"
      }
    },
    {
      "id": "log-result",
      "type": "log"
    }
  ]
}
```

### Common Design Patterns

- **Form / Webhook → RIB Validator → If Node:** Validate applicant bank details upon form submission. Route valid RIBs to payout processing and invalid RIBs to error notifications.
- **Batch Supplier List → RIB Validator (`validateRIB`) → Format RIB:** Validate batch payments, check Modulo 97 checksums, and format account numbers before exporting bank transfer files.
- **Bank Code Lookup → Customer Record Enrichment:** Use `getBankInfo` with extracted 3-digit bank codes to attach official Moroccan bank names and SWIFT codes to payee profiles.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `RIB string is required`

**Cause:** The `validateRIB` or `formatRIB` operation was called without providing a `rib` parameter.

**Solution:** Provide a 24-digit Moroccan RIB number in the `rib` field or bind it using expressions (`{{input.rib}}`).

---

#### `RIB must be 24 digits long`

**Cause:** The provided RIB input is too short or too long (must contain exactly 24 numerical digits).

**Solution:** Ensure all leading zeros are included (e.g., `"007..."`) and remove non-digit formatting characters if passed manually.

---

#### `Invalid RIB checksum key`

**Cause:** The Modulo 97 calculation failed for the 24-digit RIB sequence, indicating a typo in the bank code, counter code, account number, or control key.

**Solution:** Check the RIB number against the official bank document (Relevé d'Identité Bancaire) for transposed digits or mistyped numbers.

---

#### `Unknown Moroccan bank code`

**Cause:** The 3-digit bank code does not match any registered Moroccan financial institution.

**Solution:** Verify the bank code using `getAllBanks` or check if the bank code was entered correctly.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Banking Validator](./banking-validator.md) – Validate international IBAN and SWIFT/BIC codes
- [Function](./function.md) – Transform and map validated bank payload details
- [If](./if.md) – Branch workflows based on RIB checksum validation results
- [HTTP Request](./http-request.md) – Send validated banking data to payment APIs or ERPs

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-13 | Initial release of Moroccan RIB Validator documentation |

<!-- /SECTION: changelog -->

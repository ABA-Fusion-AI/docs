---
node_id: "vat-comply"
title: "VatComply"
description: "Validate EU VAT numbers and retrieve business details for B2B workflows."
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - vat
  - tax
  - vies
  - eu
  - finance
  - accounting
  - validation
  - b2b
related_nodes:
  - currency-converter
  - invoice-ninja
  - quickbooks
  - log
---

<!-- SECTION: header -->
# VatComply

> **Category:** Business & Commerce | **Subcategory:** Finance & Accounting | **Type:** Action Node

Validate European Union VAT numbers in real time and retrieve the associated business details for B2B sales and accounting workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **VatComply** node checks an EU VAT number against the VatComply service, which validates numbers through the official VIES system. A successful lookup can return whether the number is valid, the country code, registered name, and address when available.

### Key Features

- **VAT Validation:** Check whether an EU VAT number is valid
- **Business Details:** Retrieve the registered business name and address when returned
- **Country Identification:** Return the VAT country prefix and normalized number
- **B2B Workflows:** Support customer onboarding, invoicing, and reverse-charge review steps
- **No Credentials Required:** The workflow example contains no API key, access token, authorization header, or secret
- **Live Lookup:** Use current service data rather than relying only on local checksum validation

### Use Cases

- Validate customer VAT numbers before issuing B2B invoices
- Enrich customer or supplier records with VAT details
- Route valid and invalid VAT numbers to different accounting flows
- Support EU cross-border sales checks
- Store validation results alongside invoice or audit records

> A successful technical validation does not by itself determine the correct tax treatment for every transaction. Apply the relevant jurisdictional rules and retain any required evidence for accounting or tax review.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `vatNumber` | `string` | Yes | — | EU VAT number to validate, for example `BE0123456789` or `LU26375245` |

If `vatNumber` is not configured, the node can read the value from the incoming `input`. A direct string is treated as the VAT number; an object can provide a `vatNumber` property.

### Input Format

Use the country prefix followed by the national VAT identifier:

```text
BE0123456789
LU26375245
```

Avoid including spaces or punctuation unless the upstream workflow and API accept them. The service may normalize common formatting differences, but the country prefix must be present.

### API and Authentication

The node uses the VatComply VAT validation endpoint:

```text
GET https://api.vatcomply.com/vat?vat_number={vatNumber}
```

The provided workflow contains only `vatNumber`; no API key, access token, authorization header, or credential reference is configured. VatComply documents this endpoint as requiring no authentication.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | A VAT number string or an object containing a `vatNumber` field. Used when the configured parameter is empty. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | VAT validation result and business details returned by VatComply |

Example success response:

```json
{
  "valid": true,
  "vat_number": "BE0123456789",
  "name": "Example Company BV",
  "address": "Example Street 1, Brussels",
  "country_code": "BE"
}
```

The `name` and `address` fields may be empty or unavailable depending on the upstream VIES response.

### Error Output

Missing or malformed VAT numbers, unavailable VIES services, rate limits, and upstream API errors are routed to `error`.

```json
{
  "success": false,
  "error": "Invalid VAT number format",
  "vatNumber": "INVALID"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Validate a Belgian VAT Number

```json
{
  "vatNumber": "BE0123456789"
}
```

### Validate a Luxembourg VAT Number

```json
{
  "vatNumber": "LU26375245"
}
```

### Dynamic VAT Number from a Previous Node

Pass a VAT number directly through `input`:

```text
BE0123456789
```

Or pass a named object:

```json
{
  "vatNumber": "LU26375245"
}
```

### Branch on Validation Result

Use a conditional or Function node after VatComply to route records based on `success.valid`:

```js
return {
  vatNumber: input.vat_number,
  isValid: input.valid === true,
  companyName: input.name || null,
  country: input.country_code || null
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate EU VAT numbers for B2B workflows
```

### Common Patterns

- **Basic validation:** Manual Trigger → VatComply → Log
- **Customer onboarding:** Form/Webhook → VatComply → Conditional branch
- **Invoice review:** Invoice data → VatComply → Invoice or accounting system
- **Record enrichment:** Customer record → VatComply → Update database
- **Exception handling:** VatComply error output → Notification or manual review

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### VAT number is required

**Cause:** Neither `vatNumber` nor a usable incoming `input` value was provided.

**Solution:** Configure a VAT number, pass a string to `input`, or pass an object with a `vatNumber` property.

### Invalid VAT number format

**Cause:** The value is missing a valid country prefix or contains an invalid national identifier format.

**Solution:** Use the two-letter country prefix followed by the national VAT identifier, for example `BE0123456789`.

### VAT number is not valid

**Cause:** VIES could not validate the number, or the number is not registered as valid for intra-EU transactions.

**Solution:** Confirm the customer supplied the correct number and country, then retry or request updated registration information.

### VIES temporarily unavailable

**Cause:** A member-state service may be unavailable, busy, or over its concurrent-request limit.

**Solution:** Retry later with backoff and preserve the error details for audit or manual review.

### Missing company name or address

**Cause:** The upstream authority did not return complete business details.

**Solution:** Treat `name` and `address` as optional and do not reject an otherwise valid result solely because those fields are empty.

### Tax treatment is unclear

**Cause:** VAT-number validation alone does not resolve place-of-supply, exemption, reverse-charge, or rate rules.

**Solution:** Use the result as one input to your tax decision process and consult qualified tax or accounting guidance where required.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Currency Converter](./currency-converter.md) — Convert invoice and settlement amounts
- [Invoice Ninja](./invoice-ninja.md) — Generate and manage invoices after validation
- [QuickBooks](./quickbooks.md) — Connect validated customer data to accounting workflows
- [Log](./log.md) — Inspect validation results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial documentation |

<!-- /SECTION: changelog -->

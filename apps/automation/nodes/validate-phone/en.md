---
node_id: "validate-phone"
title: "Validate Phone"
description: "Validate phone number format."
category: "data-transformation-etl"
subcategory: "date-time"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - validation
  - phone
  - phone-number
  - e164
  - international
  - data-validation
  - data-cleaning
  - data-transformation
related_nodes:
  - schema-validate
  - validate-url
  - regex-match
  - to-string
  - html-sanitize
---

<!-- SECTION: overview -->
# Validate Phone

> **Category:** Data Transformation (ETL) &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Validate Phone** node verifies whether a given phone number conforms to standard telephone numbering formats and returns both the original and sanitized versions along with the matched format.

It supports targeted validation against specific standards (such as **US**, **International**, or **E.164**) as well as automatic format detection. Before applying regex validation, the node cleans standard formatting artifacts—such as spaces, dashes, dots, and parentheses—allowing users and webhooks to submit formatted numbers seamlessly.

```
Input Phone ("+1 (555) 234-5678")
  ↓
Clean Formatting (remove spaces, -, (, ), .) → "+15552345678"
  ↓
Validate against Format (US / International / E.164 / Auto-detect)
  ↓
Validation Successful?
  ├─ Yes → Return { valid: true, phone, cleaned, format }
  └─ No  → Throw Error ("Invalid phone number format: ...")
```

### Key Features

- **Multi-Format Support:** Validates against `us`, `international`, and `e164` standards.
- **Auto-Detection Mode:** When no format is explicitly configured, automatically tests against all supported patterns in sequence.
- **Automatic Sanitization:** Automatically removes common formatting characters (` `, `-`, `(`, `)`, `.`) while preserving the raw input.
- **Flexible Input Resolution:** Accepts direct configuration strings, upstream workflow strings, numbers, or objects with a `data` property.
- **Structured Output:** Produces clean attributes ready for CRM updates, SMS gateways, and downstream branching.

### Common Use Cases

- **Form Submission Sanitization:** Validate and normalize phone numbers captured from lead generation forms or webhooks before saving to a database or CRM (HubSpot, Salesforce).
- **Messaging Pipelines:** Ensure phone numbers match the strict E.164 format required by SMS and messaging APIs (Twilio, WhatsApp, MessageBird) before attempting to dispatch notifications.
- **Workflow Branching:** Gate automated workflows to ensure downstream communication nodes are only executed for valid, reachable contact numbers.
- **Data Migration & ETL:** Clean and standardize legacy customer phone records during bulk data import or synchronization routines.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `data` | `string` | No | — | The phone number string to validate. If empty or omitted, the node uses incoming data from upstream nodes. |
| `format` | `string` | No | — | Target validation pattern (`us`, `international`, `e164`). If omitted, the node tests all formats automatically. |

---

### Supported Formats & Patterns

The node validates cleaned numbers against regular expression patterns:

| Format Name | Regex Pattern | Description | Examples |
|-------------|---------------|-------------|----------|
| `us` | `^\+?1?[2-9]\d{2}[2-9]\d{2}\d{4}$` | North American Numbering Plan (NANP). Supports optional `+1` prefix, with area code and exchange code starting with digits 2–9. | `(555) 234-5678`<br>`+1-555-234-5678`<br>`5552345678` |
| `international` | `^\+?[1-9]\d{1,14}$` | ITU-T international numbering format. Allows 1 to 15 digits (excluding leading zeros), with an optional leading `+`. | `+212 612-345678`<br>`+44 20 7183 8750`<br>`33123456789` |
| `e164` | `^\+[1-9]\d{1,14}$` | Strict ITU-T E.164 standard. Requires a leading `+` followed by the country code and subscriber number (up to 15 digits total). | `+15552345678`<br>`+212612345678`<br>`+33612345678` |

> **Auto-Detection:** When the `format` parameter is left blank, the node tries `us`, `international`, and `e164` in order. The first pattern that matches is recorded as the `format` in the output.

---

### Input Resolution

The node resolves the phone number value in the following order:

1. **Config Parameter:** If `config.data` is provided and contains a non-empty, non-whitespace string, it is used.
2. **Upstream Workflow Data:** If `config.data` is empty, undefined, or whitespace-only, the incoming tick payload `data` is used.
3. **Type Coercion:**
   - **String:** Direct string value, trimmed.
   - **Object with `data` property:** If the input is an object containing a string property named `data` (e.g. `{ "data": "+1 (555) 000-1122" }`), it extracts that property.
   - **Other Objects:** JSON-stringified and trimmed.
   - **Primitives (Numbers, etc.):** Converted to string and trimmed.

---

### Sanitization Process

Before pattern matching, the node strips all whitespace, dashes, parentheses, and dots:

```javascript
cleaned = phoneString.replace(/[\s\-\(\)\.]/g, "");
```

For example:
- `"+1 (555) 234-5678"` → `"+15552345678"`
- `"+212 6.12.34.56.78"` → `"+212612345678"`
- `"555-234-5678"` → `"5552345678"`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` \| `number` \| `object` | Raw phone number string, numeric representation, or payload object containing a `data` field. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when validation succeeds, containing validation details and cleaned numbers. |
| `error` | `object` | Emitted if the number is missing, empty, or fails pattern validation. |

### Output Schema (`success`)

When validation succeeds, the node returns an object with the following structure:

```json
{
  "valid": true,
  "phone": "+1 (555) 234-5678",
  "cleaned": "+15552345678",
  "format": "us"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `valid` | `boolean` | Always `true` upon successful execution. |
| `phone` | `string` | The original, trimmed input string before character cleaning. |
| `cleaned` | `string` | The sanitized phone number without spaces, hyphens, periods, or parentheses. |
| `format` | `string` | The matched format identifier (`"us"`, `"international"`, or `"e164"`). |

---

### Accessing Output in Expressions

Downstream nodes can reference output values using Fusion expressions:

- **Sanitized Number (for SMS / WhatsApp API):**
  ```
  {{ outputs["Validate Phone"].cleaned }}
  ```
- **Original Phone Number:**
  ```
  {{ outputs["Validate Phone"].phone }}
  ```
- **Matched Format:**
  ```
  {{ outputs["Validate Phone"].format }}
  ```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate and Clean Phone Number
```

### How It Works

1. **Manual Trigger (`manual-trigger`):** Initiates execution with sample or form payload data.
2. **Validate Phone (`validate-phone`):** Receives the phone string, sanitizes formatting artifacts, and checks validity against the configured format (e.g. `us`).
3. **Log (`log`):** Displays the output object containing `valid: true`, the original `phone`, `cleaned` string, and matched `format`.

---

### Practical Automation Scenarios

#### Scenario 1: Lead Capture Form to CRM & SMS Dispatch

```
Webhook (New Form Lead)
  ↓
Validate Phone (Format: e164)
  ├─ [success] → Twilio Node (Send Welcome SMS to {{ outputs["Validate Phone"].cleaned }})
  │               ↓
  │             HubSpot (Create Contact with normalized phone)
  │
  └─ [error]   → Log / Error Handler (Flag record for manual review)
```

#### Scenario 2: Auto-Detect and Normalize Contact Lists

```
Google Sheets (Read Rows)
  ↓
Loop / Iterator
  ↓
Validate Phone (Format: auto-detect)
  ↓
Database / CRM (Update record with standardized .cleaned format)
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors and Solutions

#### `Data is required for phone validation`
- **Cause:** Both the `data` configuration field and the incoming workflow payload are `undefined` or `null`.
- **Solution:** Verify that an upstream node feeds data into the `Validate Phone` node's input port, or provide a default fallback value in the configuration.

#### `Cannot validate empty phone number`
- **Cause:** The resolved input evaluates to an empty string `""` or whitespace only.
- **Solution:** Check form inputs or webhook mappings to ensure phone fields are not blank before calling validation.

#### `Invalid phone number format: <value>`
- **Cause:** The cleaned phone number does not conform to the selected `format` rule or fails all auto-detection patterns.
  - For `us`: Must be 10 digits (or 11 with leading `1`), with area/exchange codes between 2–9.
  - For `e164`: Must explicitly start with `+` and country code.
  - For `international`: Must not start with `0` after removing any leading `+`.
- **Solution:** If numbers come in mixed formats without leading `+`, consider using the default auto-detect mode or preprocessing national numbers to add the appropriate international country code.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Schema Validate](../schema-validate/en.md) – Validate complex JSON objects, structures, and business rules.
- [Validate URL](../validate-url/en.md) – Validate web URLs, hostnames, and protocols.
- [Regex Match](../regex-match/en.md) – Perform custom regular expression matching on text.
- [To String](../to-string/en.md) – Convert arbitrary data types into strings.
- [HTML Sanitize](../html-sanitize/en.md) – Clean and sanitize HTML input.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Initial release with support for US, International, and E.164 formats, automatic sanitization, and auto-detection. |

<!-- /SECTION: changelog -->

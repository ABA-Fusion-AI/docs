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
  - validate-email
  - validate-url
  - schema-validate
  - regex-match
  - to-string
  - html-sanitize
---

<!-- SECTION: header -->
# Validate Phone

> **Category:** Data Transformation (ETL) | **Subcategory:** Date & Time | **Type:** Action Node

Verify phone number syntax, clean common formatting characters (spaces, dashes, parentheses, dots), and validate against **US**, **International**, and strict **E.164** standards with automatic format detection.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Validate Phone** node acts as an automated validation and sanitization gatekeeper for telephone numbers in your automation pipelines. Before forwarding customer contact numbers to messaging providers (Twilio, WhatsApp, MessageBird), saving them into CRMs (HubSpot, Salesforce), or running SMS marketing campaigns, this node ensures that the number is well-structured and conforms to telecommunication standards.

When a phone number is processed, the node cleans visual formatting artifacts (` `, `-`, `(`, `)`, `.`) and returns a structured output payload containing `valid: true`, the original `phone`, the sanitized `cleaned` string, and the `format` identifier that matched.

```
┌────────────────────────────────────────────────────────┐
│ Inbound Contact Data: "+1 (555) 234-5678"              │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Strip Formatting: [\s\-\(\)\.] ➔ "+15552345678"       │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Validate against Format (US, International, E.164, or Auto) │
└───────────────────────────┬────────────────────────────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
              [success]            [error]
                  │                   │
                  ▼                   ▼
┌───────────────────────────┐ ┌───────────────────────────┐
│ { valid: true,            │ │ Reject / Route to Error   │
│   phone: "+1 (555) 234..",│ │ Notification Flow         │
│   cleaned: "+15552345678",│ │ ("Invalid phone number")  │
│   format: "us" }          │ └───────────────────────────┘
└───────────────────────────┘
```

### Key Features

- **Multi-Format Standards:** Supports dedicated validation rules for `us`, `international`, and `e164` phone standards.
- **Smart Auto-Detection:** When no specific format is selected, the node sequentially evaluates `us`, `international`, and `e164` patterns, tagging the output with the matched format.
- **Automated Sanitization:** Strips spaces, hyphens, periods, and parentheses while preserving the original input string for reporting.
- **Flexible Input Resolution:** Accepts direct configuration strings, upstream workflow strings, numbers, or objects containing a `data` property.
- **Fail-Fast Error Handling:** Throws clear, descriptive errors when numbers are missing, empty, or fail format validation.

---

### Common Use Cases

- **Lead Intake & Webhooks:** Sanitize and validate incoming phone numbers submitted through website contact forms or lead generation funnels before database insertion.
- **SMS & Messaging Gateways:** Ensure customer numbers are sanitized into E.164 format prior to dispatching notifications via Twilio, Vonage, or WhatsApp Business API.
- **CRM Sync & Deduplication:** Clean and normalize phone numbers across customer databases to prevent duplicate entries and invalid dial records.
- **Conditional Workflow Branching:** Route valid numbers directly to automated dialers or messaging nodes while routing invalid numbers to manual review queues.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Add the **Validate Phone** node to your workflow canvas and click it to configure its parameters.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|:----:|:--------:|:-------:|-------------|
| `data` | `string` | ❌ No | — | The phone number string to validate. If empty or omitted, the node validates the incoming payload from the `input` connection. |
| `format` | `string` | ❌ No | — | Specific validation pattern to enforce (`us`, `international`, `e164`). If left blank, the node tests all formats automatically. |

---

### Parameter Details & Configuration Options

#### 1. `data` (Optional)
The target telephone number to validate.
- **Static Entry:** Enter a fixed number directly (e.g. `+1 (555) 234-5678` or `+212 612 345 678`).
- **Dynamic Expression:** Enter expressions like `outputs.Webhook.success.body.phone` or `outputs.FormTrigger.success.phone` to validate inbound payload fields dynamically.
- **Fallback:** If omitted or empty, the node automatically reads the payload passed into the `input` port from the previous node.

#### 2. `format` (Optional)
Specifies which standard to enforce during validation:

| Format Option | Regular Expression Pattern | Description | Accepted Examples |
|---------------|----------------------------|-------------|-------------------|
| `us` | `^\+?1?[2-9]\d{2}[2-9]\d{2}\d{4}$` | North American Numbering Plan (NANP). Supports 10 digits with optional `+1` prefix. Area code and exchange code must start with digits 2–9. | `(555) 234-5678`<br>`+1-555-234-5678`<br>`5552345678`<br>`15552345678` |
| `international` | `^\+?[1-9]\d{1,14}$` | ITU-T international numbering plan. Allows 1 to 15 digits (excluding leading zeros), with an optional leading `+`. | `+212 612-345678`<br>`+44 20 7183 8750`<br>`33123456789`<br>`+15552345678` |
| `e164` | `^\+[1-9]\d{1,14}$` | Strict ITU-T E.164 recommendation. Requires a leading `+` followed by the country code and subscriber number (max 15 digits). | `+15552345678`<br>`+212612345678`<br>`+33612345678`<br>`+442071838750` |

> [!TIP]
> **Auto-Detection Behavior:** If `format` is not specified, the node checks `us` first, then `international`, and finally `e164`. The first matching format is assigned to the `format` output property.

---

### Input Resolution & Coercion Rules

The node resolves and processes input data using the following logic:

1. **Config Precedence:** If `config.data` is provided as a non-empty string (after trimming), it is used as the source value.
2. **Workflow Input Fallback:** If `config.data` is `undefined`, `null`, or whitespace-only, the incoming tick payload `data` is used.
3. **Data Type Handling:**
   - **String:** Trimmed directly (`phoneString = inputData.trim()`).
   - **Object with `data` Property:** If the input is an object containing a string property named `data` (e.g. `{ "data": "+1 (555) 234-5678" }`), it extracts and trims `data`.
   - **Other Objects:** JSON-stringified and trimmed.
   - **Primitives (Numbers):** Converted to string and trimmed (`String(inputData).trim()`).
4. **Sanitization:** All occurrences of whitespace (`\s`), hyphens (`-`), parentheses (`(` and `)`), and dots (`.`) are stripped:
   ```javascript
   const cleaned = phoneString.replace(/[\s\-\(\)\.]/g, "");
   ```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input Port

| Port | Description |
|------|-------------|
| `input` | Receives data from upstream triggers or action nodes. Sourced as the phone number when `data` is not defined in the configuration panel. |

### Output Ports

| Port | Color | Description |
|------|:-----:|-------------|
| `success` | 🟢 Green | Emitted when the phone number is valid and matches the target format. Returns validation status, original string, cleaned string, and matched format. |
| `error` | 🔴 Red | Emitted when input data is missing, empty, or fails regular expression pattern validation. |

---

### Output Schema (`success`)

When validation succeeds, the node outputs a structured JSON object:

```json
{
  "valid": true,
  "phone": "+1 (555) 234-5678",
  "cleaned": "+15552345678",
  "format": "us"
}
```

| Field | Type | Description | Example |
|-------|:----:|-------------|---------|
| `valid` | `boolean` | Confirms validation was successful. Always `true` on the `success` output port. | `true` |
| `phone` | `string` | The original trimmed phone string before removing formatting characters. | `"+1 (555) 234-5678"` |
| `cleaned` | `string` | The sanitized number stripped of spaces, dashes, parentheses, and dots. Ready for SMS/API payloads. | `"+15552345678"` |
| `format` | `string` | The format identifier that validated the number (`"us"`, `"international"`, or `"e164"`). | `"us"` |

---

### How to Use Output Data in Downstream Nodes

Reference output values in downstream workflow steps using Fusion expression syntax:

| Desired Value | Expression Syntax | Typical Downstream Use Case |
|---------------|-------------------|-----------------------------|
| **Sanitized Number** | `{{ outputs["Validate Phone"].cleaned }}` | Pass to Twilio, WhatsApp, or SMS marketing nodes |
| **Original Phone** | `{{ outputs["Validate Phone"].phone }}` | Save formatted string to CRM contact notes |
| **Matched Format** | `{{ outputs["Validate Phone"].format }}` | Route by regional format using Switch or If-Else nodes |
| **Validation Flag** | `{{ outputs["Validate Phone"].valid }}` | Conditional checks in branching logic |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Step-by-Step Usage Examples

### Example 1: Validating a US Phone Number with Auto-Sanitization

Clean and validate a formatted US phone number:

**Configuration:**
- **data:** `"(415) 555-2671"`
- **format:** `"us"`

**Output (`success`):**
```json
{
  "valid": true,
  "phone": "(415) 555-2671",
  "cleaned": "4155552671",
  "format": "us"
}
```

---

### Example 2: Strict E.164 Validation for Global SMS Delivery

Ensure an international number conforms to the strict E.164 standard (requires `+` prefix):

**Configuration:**
- **data:** `"+212 612 345 678"`
- **format:** `"e164"`

**Output (`success`):**
```json
{
  "valid": true,
  "phone": "+212 612 345 678",
  "cleaned": "+212612345678",
  "format": "e164"
}
```

---

### Example 3: Automatic Format Detection

When incoming contact lists have mixed domestic and international formats:

**Configuration:**
- **data:** `"+44 20 7183 8750"`
- **format:** `""` *(Leave empty for auto-detection)*

**Output (`success`):**
```json
{
  "valid": true,
  "phone": "+44 20 7183 8750",
  "cleaned": "+442071838750",
  "format": "international"
}
```

---

### Example 4: Extracting from Nested Object Payloads

When receiving form payloads wrapped in an object like `{ "data": "+1 (555) 000-1122" }`:

- The node automatically extracts the string value under `data`.
- Returns `{ "valid": true, "phone": "+1 (555) 000-1122", "cleaned": "+15550001122", "format": "us" }`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate and Clean Phone Number
```

### How It Works

1. **Manual Trigger (`manual-trigger`):** Triggers the workflow execution manually with sample contact payloads.
2. **Validate Phone (`validate-phone`):** Receives the phone string, strips visual delimiters (` `, `-`, `(`, `)`, `.`), and tests against the configured format standard.
3. **Log (`log`):** Displays the output object containing `valid: true`, the original `phone`, `cleaned` string, and matched `format`.

---

### Real-World Automation Scenarios

#### Scenario 1: Webhook Lead Capture to CRM and SMS Welcome Message

```
Webhook (New Lead Form)
  ↓
Validate Phone (Format: e164)
  ├─ [success] ➔ Twilio (Send SMS to {{ outputs["Validate Phone"].cleaned }})
  │               ↓
  │             HubSpot (Create Contact with normalized phone)
  │
  └─ [error]   ➔ Slack / Notification (Alert team of invalid lead submission)
```

#### Scenario 2: Batch Cleansing Customer Phone Records

```
Database / Google Sheets (Fetch Contacts)
  ↓
Loop / Iterator
  ↓
Validate Phone (Format: auto-detect)
  ├─ [success] ➔ Database Update (Store .cleaned phone)
  └─ [error]   ➔ Flag Record for Manual Review
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors and Solutions

#### `Data is required for phone validation`
- **Cause:** Both `config.data` and incoming payload data are `undefined` or `null`.
- **Solution:** Verify that an upstream node is connected and passing data to the input port, or provide a default fallback value in the configuration.

#### `Cannot validate empty phone number`
- **Cause:** The input string contains only whitespace or is an empty string `""`.
- **Solution:** Check your form or webhook mapping to ensure phone fields are not blank before executing validation.

#### `Invalid phone number format: <value>`
- **Cause:** The sanitized number fails regular expression matching:
  - **For `us`:** Must be 10 digits (or 11 digits starting with `1`). Area code and central office code cannot start with `0` or `1`.
  - **For `e164`:** Must explicitly begin with a `+` symbol followed by 1 to 14 digits (total max 15 digits).
  - **For `international`:** Must not start with `0` after stripping characters.
- **Solution:** If numbers arrive without a leading `+` for international destinations, use auto-detection or a **Function** node upstream to prepend the appropriate country code.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Validate Email](../validate-email/en.md) – Validate email address structure and syntax.
- [Validate URL](../validate-url/en.md) – Validate web addresses, protocols, hostnames, and URLs.
- [Schema Validate](../schema-validate/en.md) – Validate structured JSON objects using JSON Schema or Laravel rules.
- [Regex Match](../regex-match/en.md) – Test and match custom regular expression patterns against strings.
- [To String](../to-string/en.md) – Convert arbitrary data types into strings.
- [HTML Sanitize](../html-sanitize/en.md) – Clean and sanitize HTML input.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Initial release with support for US, International, and E.164 formats, automated sanitization, and auto-detection. |

<!-- /SECTION: changelog -->

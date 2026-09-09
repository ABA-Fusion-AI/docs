---
node_id: "validate-email"
title: "Validate Email"
description: "Validate email format."
category: "communication-messaging"
subcategory: "email"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - validation
  - email
  - email-validation
  - regex
  - communication
  - data-validation
  - data-cleaning
related_nodes:
  - validate-phone
  - validate-url
  - schema-validate
  - smtp
  - sendgrid
  - gmail
---

<!-- SECTION: overview -->
# Validate Email

> **Category:** Communication & Messaging &nbsp;&nbsp;|&nbsp;&nbsp;**Subcategory:** Email &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Validate Email** node verifies whether a given email address conforms to standard email format syntax (`username@domain.tld`) using regular expression pattern matching.

It provides a lightweight and reliable validation step in automation workflows, ensuring downstream communication nodes (such as SMTP, SendGrid, Mailchimp, or CRM integrations) only receive properly structured email addresses, preventing delivery errors and bouncebacks.

```
Input Email ("  user@example.com  ")
  ↓
Extract & Trim String ("user@example.com")
  ↓
Pattern Validation (/^[^\s@]+@[^\s@]+\.[^\s@]+$/)
  ↓
Validation Successful?
  ├─ Yes → Return { valid: true, email: "user@example.com" }
  └─ No  → Throw Error ("Invalid email format: ...")
```

### Key Features

- **Regex Format Verification:** Validates against the universal email pattern `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`.
- **Automatic Trimming:** Strips leading and trailing whitespace automatically before evaluation.
- **Flexible Input Resolution:** Sourced from node configuration or dynamically from incoming workflow payloads (strings, numbers, or objects with a `data` key).
- **Error Guarding:** Blocks malformed email strings from propagating to critical transactional email or marketing services.

### Common Use Cases

- **Form Submission Verification:** Validate email addresses submitted through contact forms, lead funnels, or webhooks before saving them to a database or marketing list.
- **Pre-Send Verification:** Gate automated email pipelines (SMTP, Gmail, SendGrid) to prevent attempts to send messages to invalid email addresses.
- **Data Cleansing & Enrichment:** Filter or flag invalid customer contact records during CRM migrations and automated synchronization routines.
- **Conditional Branching:** Pair with an `if-else` or error-handling route to prompt users to correct their email address when invalid.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `data` | `string` | No | — | The email address to validate. If empty or omitted, the node falls back to incoming data from upstream nodes. |

---

### Validation Rule & Regex

The node validates the trimmed string against the standard email regular expression:

```javascript
/^[^\s@]+@[^\s@]+\.[^\s@]+$/
```

#### Rule Breakdown:

- `^[^\s@]+`: Begins with one or more characters that are **not** whitespace and **not** `@` (the local/user part).
- `@`: Exactly one `@` symbol separating user and domain.
- `[^\s@]+`: One or more domain name characters (excluding whitespace and `@`).
- `\.`: A literal dot `.` separating domain name and extension.
- `[^\s@]+$`: One or more characters representing the top-level domain (TLD) ending the string.

#### Examples:

| Input Value | Evaluation | Result |
|-------------|:----------:|--------|
| `user@example.com` | Match | Valid (`true`) |
| `john.doe+newsletter@company.co.uk` | Match | Valid (`true`) |
| `contact@subdomain.domain.org` | Match | Valid (`true`) |
| `plainaddress` | No `@` symbol | Error (`Invalid email format`) |
| `@missing-local.com` | Missing local username | Error (`Invalid email format`) |
| `user@.missingdomain` | Missing domain name | Error (`Invalid email format`) |
| `user@domain` | Missing top-level domain (`.com`) | Error (`Invalid email format`) |
| `user with space@domain.com` | Contains space inside | Error (`Invalid email format`) |

---

### Input Resolution

The node resolves the target email value in the following precedence:

1. **Config Parameter:** If `config.data` is provided as a non-empty string (`trimmed !== ""`) or non-string value, it is used directly.
2. **Upstream Workflow Data:** If `config.data` is empty string, `undefined`, or `null`, the incoming tick payload `data` is used.
3. **Object Extraction:**
   - If the input is an object containing a `data` key (e.g. `{ "data": "user@example.com" }`), it extracts that property.
   - If the `data` property is a string, it trims it directly. Otherwise, other objects are JSON-stringified.
4. **Primitives:** Numbers or other primitive types are converted to strings via `String(inputData).trim()`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` \| `object` \| `number` | Incoming email address string or payload object containing a `data` property. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when validation succeeds, containing confirmation and the trimmed email address. |
| `error` | `object` | Emitted when input data is missing or fails email format validation. |

### Output Schema (`success`)

When validation succeeds, the node outputs an object:

```json
{
  "valid": true,
  "email": "user@example.com"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `valid` | `boolean` | `true` when the email matches the valid format. |
| `email` | `string` | The validated and trimmed email address. |

---

### Accessing Output in Expressions

Downstream workflow nodes can access the validated email via Fusion expressions:

- **Email Address:**
  ```
  {{ outputs["Validate Email"].email }}
  ```
- **Validation Flag:**
  ```
  {{ outputs["Validate Email"].valid }}
  ```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate Email Address
```

### How It Works

1. **Manual Trigger (`manual-trigger`):** Starts workflow execution with sample data or form parameters.
2. **Validate Email (`validate-email`):** Resolves the email string, trims whitespace, and applies the email regex pattern.
3. **Log (`log`):** Receives the `success` output and outputs `{ "valid": true, "email": "user@example.com" }` to the execution log.

---

### Common Automation Patterns

#### 1. Form Webhook to Marketing CRM & Confirmation Email

```
Webhook (New Lead)
  ↓
Validate Email
  ├─ [success] → SendGrid / SMTP (Send Confirmation Email to {{ outputs["Validate Email"].email }})
  │               ↓
  │             Mailchimp / Brevo (Add subscriber)
  │
  └─ [error]   → Slack / Discord (Alert team about invalid submission)
```

#### 2. Batch Validation in Array Loops

```
Fetch Leads (Database / Google Sheets)
  ↓
Loop / Iterator
  ↓
Validate Email
  ├─ [success] → CRM Add Contact
  └─ [error]   → Log Invalid Record
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors and Solutions

#### `Data is required for email validation`
- **Cause:** Both the `data` configuration field and the incoming workflow payload are `undefined` or `null`.
- **Solution:** Verify that an upstream node feeds data into the `Validate Email` node's input port, or provide a default value in the configuration panel.

#### `Invalid email format: <value>`
- **Cause:** The input string does not conform to `username@domain.extension`.
  - Missing `@` symbol or domain dot (e.g. `user@com` or `user.example.com`).
  - Contains unescaped internal spaces (e.g. `user name@example.com`).
  - Empty string `""` provided as input.
- **Solution:** Ensure the source field mapped into the node contains a complete email address. Use a fallback or conditional check if optional form fields may be empty.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Validate Phone](../validate-phone/en.md) – Validate and clean telephone numbers across US, International, and E.164 formats.
- [Validate URL](../validate-url/en.md) – Validate web addresses, URLs, hostnames, and protocols.
- [Schema Validate](../schema-validate/en.md) – Validate complex JSON payloads with JSON Schema or Laravel-style rules.
- [SMTP: Send Email](../smtp/en.md) – Send transactional emails using custom SMTP credentials.
- [SendGrid](../sendgrid/en.md) – Deliver marketing and transactional emails via SendGrid API.
- [Gmail](../gmail/en.md) – Send and receive emails through Google Workspace / Gmail.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Initial release with regex-based email format validation and flexible input resolution. |

<!-- /SECTION: changelog -->

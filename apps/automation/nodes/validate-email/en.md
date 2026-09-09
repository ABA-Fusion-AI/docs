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

<!-- SECTION: header -->
# Validate Email

> **Category:** Communication & Messaging | **Subcategory:** Email | **Type:** Action Node

Verify the structural syntax of email addresses using standard regex pattern matching — ensuring only correctly formatted addresses reach downstream communication nodes, CRM systems, and marketing platforms.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Validate Email** node acts as an automated quality gate for email addresses within your automation pipelines. Before forwarding contact addresses to transactional email services (SMTP, SendGrid, Mailchimp, Brevo), saving them to databases, or adding them to CRM contact lists, this node confirms that the address conforms to the standard `username@domain.tld` structure.

When validation succeeds, the node returns a structured payload containing `valid: true` and the trimmed, normalized email string — ready to be referenced by downstream nodes. When it fails, it emits on the `error` port so invalid submissions can be gracefully handled without crashing the workflow.

```
┌────────────────────────────────────────────────────────┐
│ Inbound Email: "  user@example.com  "                  │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Extract & Trim: "user@example.com"                     │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Regex Test: /^[^\s@]+@[^\s@]+\.[^\s@]+$/              │
└───────────────────────────┬────────────────────────────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
              [success]            [error]
                  │                   │
                  ▼                   ▼
┌───────────────────────────┐ ┌───────────────────────────┐
│ { valid: true,            │ │ Reject / Route to Error   │
│   email: "user@example"   │ │ Handler / Alert Flow      │
│   "@example.com" }        │ │ ("Invalid email format")  │
└───────────────────────────┘ └───────────────────────────┘
```

### Key Features

- **RFC-Aligned Regex Check:** Validates the `local@domain.tld` format using a universal pattern that handles subdomains, plus-addressing, and multi-level TLDs.
- **Automatic Whitespace Trimming:** Leading and trailing spaces are stripped automatically before evaluation (e.g. `" user@example.com "` becomes `"user@example.com"`).
- **Flexible Input Resolution:** Accepts email strings from node configuration, upstream payload strings, numbers, or objects containing a `data` property — resolved in a clear precedence order.
- **Fail-Fast Error Branching:** Emits on the red `error` port with a clear message when data is missing or invalid, enabling graceful fallback routing.

---

### Common Use Cases

- **Lead Form & Webhook Validation:** Screen email addresses submitted through contact forms or webhooks before inserting them into a CRM or marketing list.
- **Pre-Send Email Gating:** Prevent transactional email dispatch to malformed addresses — protecting your sender reputation and reducing bounce rates.
- **Data Migration & Cleansing:** Validate and flag invalid email records during bulk import jobs or CRM synchronization routines.
- **Conditional Branching:** Route invalid submissions to alert channels (Slack, Discord) or a manual review queue while valid ones continue downstream.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Add the **Validate Email** node to your workflow canvas and click it to open the configuration panel.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|:----:|:--------:|:-------:|-------------|
| `data` | `string` | ❌ No | — | The email address to validate. If empty or omitted, the node validates the incoming payload from the `input` connection. |

---

### Parameter Details

#### `data` (Optional)
The target email address you want to validate.
- **Static Entry:** Enter a complete email directly (e.g. `user@example.com`).
- **Dynamic Expression:** Click **Expression** and type an expression like `outputs.Webhook.success.body.email` or `outputs.FormTrigger.success.email` to validate email addresses from upstream nodes dynamically.
- **Fallback:** If left empty, the node automatically reads the incoming data payload from the `input` port connection.

> [!IMPORTANT]
> **Full Email Format Required:** A valid email must include a local username, an `@` symbol, a domain name, a dot, and a top-level domain (e.g. `user@example.com`). Strings like `user@domain` without a TLD or `user.example.com` without an `@` will fail validation.

---

### Validation Rule & Regex Pattern

The node validates the trimmed input against the following regular expression:

```javascript
/^[^\s@]+@[^\s@]+\.[^\s@]+$/
```

#### Pattern Breakdown

| Segment | Meaning |
|---------|---------|
| `^[^\s@]+` | Local username: one or more characters that are **not** a space (`\s`) or `@`. |
| `@` | Exactly **one** `@` symbol separating username and domain. |
| `[^\s@]+` | Domain name: one or more characters that are **not** a space or `@`. |
| `\.` | A literal **dot** (`.`) separating domain name from TLD. |
| `[^\s@]+$` | Top-level domain: one or more characters that are **not** a space or `@`. |

#### Validation Examples

| Input Value | Evaluation | Result |
|-------------|:----------:|--------|
| `user@gmail.com` | ✅ Match | Valid (`true`) |
| `contact@entreprise.ma` | ✅ Match | Valid (`true`) |
| `ahmed.benali.dev@domain.com` | ✅ Match | Valid (`true`) |
| `john.doe+newsletter@gmail.com` | ✅ Match — plus-addressing supported | Valid (`true`) |
| `support@billing.cloud.aws.com` | ✅ Match — multi-level subdomain | Valid (`true`) |
| `info@company.co.uk` | ✅ Match — multi-part TLD | Valid (`true`) |
| `plainaddress` | ❌ No `@` or dot | Error: Invalid email format |
| `@missing-local.com` | ❌ Empty local username | Error: Invalid email format |
| `user@domain` | ❌ Missing TLD | Error: Invalid email format |
| `user with space@domain.com` | ❌ Space in local part | Error: Invalid email format |
| `user@.missingdomain.com` | ❌ Empty domain segment | Error: Invalid email format |

---

### Input Resolution & Coercion Rules

The node determines the email source using the following precedence order:

1. **Config Precedence:** If `config.data` is a non-empty string (after trimming), it is used directly.
2. **Non-String Config Fallback:** If `config.data` exists but is not a string, it is used as-is.
3. **Workflow Input Fallback:** If `config.data` is `undefined`, `null`, or whitespace-only:
   - If the incoming `data` payload is an **object with a `data` property** (e.g. `{ "data": "user@example.com" }`), that property is extracted.
   - Otherwise, the raw `data` payload is used directly.
4. **Type Coercion:**
   - **String:** Trimmed and used directly.
   - **Object with `data` string property:** Extracted and trimmed.
   - **Other objects:** JSON-stringified and trimmed.
   - **Primitives (numbers, booleans):** Converted via `String(inputData).trim()`.

> [!TIP]
> To pass an email address from a **Function** node into **Validate Email** using an expression, have the Function return `{ "test": "info@company.co.uk" }` and set `data` in configuration to the expression `outputs.Function.success.test`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input Port

| Port | Description |
|------|-------------|
| `input` | Receives data from upstream triggers or action nodes. Sourced as the email address when `data` is not defined in the configuration panel. |

### Output Ports

| Port | Color | Description |
|------|:-----:|-------------|
| `success` | 🟢 Green | Emitted when the email address passes format validation. Returns `valid: true` and the trimmed email string. |
| `error` | 🔴 Red | Emitted when input data is missing, empty, or fails the email format regex. |

---

### Output Schema (`success`)

When validation succeeds, the node returns:

```json
{
  "valid": true,
  "email": "user@example.com"
}
```

| Field | Type | Description | Example |
|-------|:----:|-------------|---------|
| `valid` | `boolean` | Always `true` on the `success` output port. | `true` |
| `email` | `string` | The validated, trimmed email address ready for downstream use. | `"user@example.com"` |

---

### How to Use Output Data in Downstream Nodes

Reference the validated email in downstream workflow steps using Fusion expressions:

| Desired Value | Expression Syntax | Typical Downstream Use Case |
|---------------|-------------------|-----------------------------|
| **Email Address** | `{{ outputs["Validate Email"].email }}` | Pass to SMTP, SendGrid, Gmail, or CRM nodes |
| **Validation Flag** | `{{ outputs["Validate Email"].valid }}` | Conditional checks in branching logic |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Step-by-Step Usage Examples

### Example 1: Simple Email Address Validation

Validate a standard email address entered in the configuration panel:

**Configuration:**
- **data:** `"user@gmail.com"`

**Output (`success`):**
```json
{
  "valid": true,
  "email": "user@gmail.com"
}
```

---

### Example 2: Plus-Addressed Email (Tag Routing)

Validate an email with a plus-tag commonly used by Gmail for inbox filtering:

**Configuration:**
- **data:** `"john.doe+newsletter@gmail.com"`

**Output (`success`):**
```json
{
  "valid": true,
  "email": "john.doe+newsletter@gmail.com"
}
```

---

### Example 3: Multi-Level Subdomain & Multi-Part TLD

Validate an enterprise email with a subdomain and a country-code TLD:

**Configuration:**
- **data:** `"support@billing.cloud.aws.com"` or `"info@company.co.uk"`

**Output (`success`):**
```json
{
  "valid": true,
  "email": "support@billing.cloud.aws.com"
}
```

---

### Example 4: Automatic Whitespace Trimming

Validate an email with accidental leading spaces (e.g. copy-pasted from a spreadsheet):

**Configuration:**
- **data:** `" info@company.co.uk"` *(note leading space)*

**Output (`success`):**
```json
{
  "valid": true,
  "email": "info@company.co.uk"
}
```
> The node trims whitespace before applying regex validation.

---

### Example 5: Validating Upstream Function Output (Dynamic Expression)

When a **Function** node returns a structured object, use an expression to extract the email:

**Function Node Code:**
```javascript
const test = "info@company.co.uk";
return { test };
```

**Validate Email Configuration:**
- **data:** *(Expression mode)* `outputs.Function.success.test`

**Output (`success`):**
```json
{
  "valid": true,
  "email": "info@company.co.uk"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Validate Email Address — Multiple Scenarios
```

### What the Workflow Demonstrates

The included example workflow showcases **7 real-world validation scenarios** arranged vertically on the canvas:

| Scenario | Email Tested | Validates |
|----------|-------------|-----------|
| 1 | `user@gmail.com` | Basic standard email |
| 2 | `contact@entreprise.ma` | Moroccan domain TLD |
| 3 | `ahmed.benali.dev@domain.com` | Dotted local name |
| 4 | `john.doe+newsletter@gmail.com` | Plus-addressed email |
| 5 | `support@billing.cloud.aws.com` | Multi-level subdomain |
| 6 | `info@company.co.uk` | Leading space (auto-trimmed) |
| 7 | Dynamic from **Function** node | Expression-bound input (`outputs.Function.success.test`) |

Each scenario consists of a **Manual Trigger** → **Validate Email** pair, making each independently executable for rapid testing.

---

### Common Automation Patterns

#### 1. Webhook Lead Capture to CRM & Confirmation Email

```
Webhook (New Lead Form Submission)
  ↓
Validate Email
  ├─ [success] ➔ SendGrid / SMTP (Send confirmation to {{ outputs["Validate Email"].email }})
  │               ↓
  │             Mailchimp / Brevo (Add to subscriber list)
  │
  └─ [error]   ➔ Slack / Discord (Alert: invalid email submitted)
```

#### 2. Batch Contact List Cleansing

```
Google Sheets / Database (Fetch Contact Records)
  ↓
Loop / Iterator
  ↓
Validate Email
  ├─ [success] ➔ CRM (Add or update valid contact)
  └─ [error]   ➔ Flagging System (Mark record for manual review)
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Errors and Solutions

#### `Data is required for email validation`
- **Cause:** Both `config.data` and the incoming workflow payload are `undefined` or `null`.
- **Solution:** Ensure an upstream node is connected to the `input` port and is passing data, or set a default email value directly in the configuration panel.

#### `Invalid email format: <value>`
- **Cause:** The trimmed input string fails the regex `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`:
  - Missing `@` symbol entirely (e.g. `userexample.com`).
  - Missing TLD or domain dot (e.g. `user@domain`).
  - Contains internal whitespace (e.g. `user name@domain.com`).
  - Empty string `""` after trimming.
- **Solution:** Check that the source field holds a complete email address. If the value may be empty, add a conditional check or default upstream, or handle the `error` port gracefully to avoid blocking your workflow.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Validate Phone](../validate-phone/en.md) – Validate and clean telephone numbers across US, International, and E.164 formats.
- [Validate URL](../validate-url/en.md) – Validate web addresses, URLs, hostnames, and enforce protocol allowlists.
- [Schema Validate](../schema-validate/en.md) – Validate complex JSON payloads with JSON Schema or Laravel-style rules.
- [SMTP: Send Email](../smtp/en.md) – Send transactional emails using custom SMTP server credentials.
- [SendGrid](../sendgrid/en.md) – Deliver marketing and transactional emails via the SendGrid API.
- [Gmail](../gmail/en.md) – Send and receive emails through Google Workspace or Gmail.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Initial release with regex-based email format validation, automatic whitespace trimming, flexible input resolution, and multi-scenario example workflow. |

<!-- /SECTION: changelog -->

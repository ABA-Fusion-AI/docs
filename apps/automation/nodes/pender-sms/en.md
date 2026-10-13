---
node_id: "pender-sms"
title: "PENDER SMS - Send Single Message"
description: "Send a single SMS message using PENDER/Piktar SMS API via linked Android devices or credit gateways."
category: "peer-only"
subcategory: "Communication"
version: "1.0.0"
language: "en"
last_updated: "2026-10-13"
author: "Fusion Team"
tags:
  - pender
  - piktar
  - sms
  - messaging
  - mobile
  - peer-only
related_nodes:
  - messagebird
  - seven-io
  - twilio
  - http-request
---

<!-- SECTION: overview -->
# PENDER SMS - Send Single Message

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send SMS messages through the **PENDER / Piktar SMS Gateway API** (`https://sms.piktar.cloud/api`). The node supports two sending modes:
- **`devices` mode:** Sends messages through your linked Android mobile hardware running the PENDER/Piktar gateway app (supports Dual-SIM selection).
- **`credits` mode:** Routes messages via centralized provider gateways using purchased platform credits.

The node features automatic international phone number normalization (e.g., standardizing local Moroccan numbers starting with `0` or 9 digits to `+212`, or formatting numbers with international prefixes).

### Use Cases

- **Transactional Alerts & OTPs:** Send one-time passwords, verification codes, or order updates instantly to customer phones.
- **Workflow Notifications:** Notify administrators or customers upon form submission, lead capture, or payment status changes.
- **Hardware-based SMS routing:** Leverage connected Android SIM cards for low-cost domestic mobile messaging plans.
- **Credit-based Gateway routing:** Dispatch broadcast or priority SMS messages via external gateway partners.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `secret` | `string` | Yes | — | PENDER / Piktar API secret key. Supports expressions. |
| `baseUrl` | `url` | No | `https://sms.piktar.cloud/api` | API Base URL. |
| `mode` | `enum` | Yes | `devices` | Sending mode: `devices` (linked Android hardware) or `credits` (partner gateway credits). |
| `phone` | `string` | Yes | — | Recipient phone number. Automatically normalized to international format (e.g. `+2126...`). Supports expressions. |
| `message` | `string` | Yes | — | Text message content to send. Supports expressions. |
| `device` | `string` | For `devices` mode | — | Unique Device ID of the linked Android phone. |
| `sim` | `number` | For `devices` mode | — | SIM slot index to use on the device (`1` for SIM 1, `2` for SIM 2). |
| `priority` | `number` | No | `1` | Message delivery priority for devices mode (`1` for normal, `2` for high). |
| `gateway` | `string` / `number` | For `credits` mode | — | Gateway ID or name to route message via credits. |
| `shortener` | `number` | No | — | Optional URL shortener identifier or flag. |

### Sending Modes & Requirements

| Mode | Target Infrastructure | Required Fields |
|------|------------------------|-----------------|
| `devices` | Linked Android Smartphone | `secret`, `phone`, `message`, `device`, `sim` (1 or 2) |
| `credits` | Platform Gateway Provider | `secret`, `phone`, `message`, `gateway` |

### Phone Number Normalization

Phone numbers are automatically sanitized and normalized to E.164 international format:
- Strips spaces, hyphens, and non-digit characters (preserves `+`).
- Local Moroccan numbers starting with `0` (e.g. `0612345678`) are automatically converted to `+212612345678`.
- 9-digit numbers without leading 0 are prefixed with `+212`.
- Any other number missing a leading `+` has `+` prepended.

### Authentication & API Key

1. Log in to your **PENDER / Piktar SMS Dashboard** (`https://sms.piktar.cloud`).
2. Navigate to **API Keys / Settings**.
3. Generate or copy your **Secret Key**.
4. Paste the key into the `secret` field (or store it in Fusion Secrets).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream variables via expressions in `phone` and `message`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returns operation result payload. Contains `success: true` or `success: false` with error message. |
| `error` | `Error` | Emitted on uncaught runtime errors. |

### Output Examples

#### Successful SMS Delivery (`devices` mode)

```json
{
  "success": true,
  "smsId": 894521,
  "status": "sent",
  "phone": "+212612345678",
  "message": "Your verification code is 482910.",
  "mode": "devices",
  "response": {
    "status": 200,
    "message": "The SMS has been sent successfully.",
    "id": 894521
  }
}
```

#### Successful SMS Delivery (`credits` mode)

```json
{
  "success": true,
  "smsId": 901243,
  "status": "sent",
  "phone": "+212612345678",
  "message": "Order #4891 has been confirmed.",
  "mode": "credits",
  "response": {
    "status": 200,
    "message": "The SMS has been sent successfully.",
    "id": 901243
  }
}
```

#### Validation Error

```json
{
  "success": false,
  "error": "Device ID is required when mode is \"devices\"",
  "phone": "+212612345678",
  "message": "Your verification code is 482910."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send SMS notification with PENDER SMS
```

### How it flows

1. **Manual Trigger:** Initiates the workflow with contact phone and message text.
2. **PENDER SMS Node:** Automatically normalizes the phone number and posts form data to Piktar/PENDER API.
3. **Log Node:** Displays the delivery confirmation and message ID.

### Example Configuration

```json
{
  "secret": "{{ secrets.PENDER_API_SECRET }}",
  "baseUrl": "https://sms.piktar.cloud/api",
  "mode": "devices",
  "phone": "{{ input.phone }}",
  "message": "Hello {{ input.customerName }}, your appointment is confirmed.",
  "device": "android_device_01",
  "sim": 1,
  "priority": 1
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your PENDER `secret` key in Fusion's **Secrets** vault. Do not commit hardcoded secrets or plain-text tokens into version control.

- Always verify that the linked Android device has adequate battery and stable cellular connectivity.
- Limit SMS broadcasting frequency to respect mobile carrier anti-spam and throttling regulations.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Device ID is required when mode is "devices"`
- **Cause:** `mode` is set to `devices` but the `device` field was left empty.
- **Solution:** Provide the unique device ID registered in your PENDER/Piktar console.

#### `Sim slot number must be 1 or 2`
- **Cause:** The `sim` slot parameter was missing or set to a number other than `1` or `2`.
- **Solution:** Set `sim` to `1` (primary SIM) or `2` (secondary SIM).

#### `Gateway is required when mode is "credits"`
- **Cause:** `mode` is set to `credits` but no `gateway` ID was supplied.
- **Solution:** Specify the gateway identifier configured in your Piktar account.

#### `PENDER API error: 401 - Unauthorized / Invalid Secret`
- **Cause:** The `secret` token is invalid, expired, or incorrectly formatted.
- **Solution:** Verify your secret key in the Piktar/PENDER portal.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-13 | Initial documentation: parameter reference, modes, phone normalization, output schemas, and workflow integration |

<!-- /SECTION: changelog -->


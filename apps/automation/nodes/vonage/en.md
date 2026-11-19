---
node_id: "vonage"
title: "Vonage"
description: "Send SMS, make voice calls, check the account balance and verify phone numbers with the Vonage (Nexmo) APIs."
category: "peer-only"
subcategory: "Communication"
version: "1.0.0"
language: "en"
last_updated: "2026-11-19"
author: "Fusion Team"
tags:
  - vonage
  - nexmo
  - sms
  - voice
  - verify
  - otp
related_nodes:
  - messagebird
  - seven
  - infobip-sms
---

<!-- SECTION: overview -->
# Vonage

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send **SMS**, place **voice calls**, read the **account balance** and run **phone number verification (OTP)** with **Vonage** (formerly Nexmo). The node offers **5 operations**:

| Resource | Operations |
|----------|------------|
| Messages | `sendSms` |
| Voice | `makeCall` |
| Verify | `verifyNumber`, `checkVerification` |
| Account | `getBalance` |

### Use Cases

- **Notifications:** Send order updates and alerts by SMS.
- **Two-factor authentication:** Send a verification code to a user and check the code they enter.
- **Voice alerts:** Call an on-call engineer and play a spoken message when a critical alert fires.
- **Budget monitoring:** Check the account balance before a large sending campaign.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | Operation to perform (default `sendSms`). |
| `apiKey` | `string` | Yes | Vonage API key. |
| `apiSecret` | `string` | Yes | Vonage API secret. |
| `jwt` | `string` | For `makeCall` | JWT generated from a Vonage application (Voice API). |
| `from` | `string` | For `sendSms`, `makeCall` | Sender. For SMS: a name (e.g. `Vonage APIs`) or a number. For calls: a phone number (digits only or `+` prefix, up to 15 digits). |
| `to` | `string` | For `sendSms`, `makeCall`, `verifyNumber` | Recipient number in international format (e.g. `212612345678`). |
| `text` | `string` | For `sendSms` | SMS text. |
| `answerUrl` | `string` | For `makeCall` | URL returning the NCCO (call script) to run when the call is answered. |
| `brand` | `string` | No | Brand name shown in the verification message (default `Fusion`). |
| `requestId` | `string` | For `checkVerification` | `request_id` returned by `verifyNumber`. |
| `code` | `string` | For `checkVerification` | Code received by the user. |

> All parameters are shown for every operation in the builder; only the ones listed in the table below are used.

### Operations

| Operation | Endpoint | Auth | Required |
|-----------|----------|------|----------|
| `sendSms` | `POST https://rest.nexmo.com/sms/json` | API key + secret | `from`, `to`, `text` |
| `makeCall` | `POST https://api.nexmo.com/v1/calls` | JWT (Bearer) | `jwt`, `from`, `to`, `answerUrl` |
| `getBalance` | `GET https://rest.nexmo.com/account/get-balance` | API key + secret | — |
| `verifyNumber` | `POST https://api.nexmo.com/verify/json` | API key + secret | `to` (optional `brand`) |
| `checkVerification` | `POST https://api.nexmo.com/verify/check/json` | API key + secret | `requestId`, `code` |

### Authentication

#### API key and secret
1. Log in to the **Vonage API Dashboard**.
2. Copy the **API key** and **API secret** shown on the home page.
3. Paste them into `apiKey` and `apiSecret` (or store them in Fusion Secrets).

#### JWT (for `makeCall`)
1. In the dashboard, open **Applications → Create a new application**.
2. Click **Generate public and private key** (a `private.key` file is downloaded) and enable **Voice**.
3. Save, then copy the **Application ID**.
4. Generate a JWT from the Application ID and the private key, with the **Vonage JWT generator** on developer.vonage.com or the CLI:
   ```bash
   npm i -g @vonage/cli
   vonage jwt --app_id=<APP_ID> --private_key=./private.key
   ```
5. Paste the token into `jwt`.

> A JWT expires (often after 15 minutes). Generate a new one when calls fail with `401`.

#### Answer URL (for `makeCall`)
`answerUrl` must return an **NCCO** (JSON list of call actions). For a quick test, use Vonage's text-to-speech example:
`https://raw.githubusercontent.com/nexmo-community/ncco-examples/gh-pages/text-to-speech.json`

> **Trial accounts** can only send SMS and calls to the numbers added under **Test numbers** in the dashboard.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | JSON response of Vonage. |
| `error` | `Error` | Emitted on missing fields or HTTP errors (`Vonage API Error: <status> <statusText> <body>`). |

> `sendSms`, `verifyNumber` and `checkVerification` answer with HTTP `200` even when Vonage refuses the request: check the `status` field (`"0"` = success) and `error_text` in the success output.

### Output Examples

#### `sendSms`

```json
{
  "message-count": "1",
  "messages": [
    {
      "to": "212612345678",
      "message-id": "6918e69c-8923-42ed-b63d-ebb92b94688a",
      "status": "0",
      "remaining-balance": "0.30573000",
      "message-price": "0.20258000",
      "network": "60400"
    }
  ]
}
```

#### `makeCall`

```json
{
  "status": "started",
  "direction": "outbound",
  "uuid": "c48c2eb0-2107-48fc-9c34-4ef2dc9635bc",
  "conversation_uuid": "CON-0d8268b3-7720-42a7-866a-aaee36286d4f"
}
```

#### `getBalance`

```json
{ "value": 0.30573, "autoReload": false }
```

#### `verifyNumber`

```json
{ "request_id": "12c7d3f6b1e846d39fa8f498093bef0a", "status": "0" }
```

#### `checkVerification` (expired or already verified request)

```json
{
  "status": "6",
  "error_text": "The Nexmo platform was unable to process this message for the following reason: Request '...' was not found or it has been verified already."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Vonage in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Vonage Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Send an SMS

```json
{
  "operation": "sendSms",
  "apiKey": "{{ secrets.VONAGE_API_KEY }}",
  "apiSecret": "{{ secrets.VONAGE_API_SECRET }}",
  "from": "Fusion",
  "to": "212612345678",
  "text": "Your order {{ input.orderId }} has shipped."
}
```

#### Voice call

```json
{
  "operation": "makeCall",
  "apiKey": "{{ secrets.VONAGE_API_KEY }}",
  "apiSecret": "{{ secrets.VONAGE_API_SECRET }}",
  "jwt": "{{ secrets.VONAGE_JWT }}",
  "from": "212612345678",
  "to": "212612345678",
  "answerUrl": "https://example.com/ncco.json"
}
```

#### Verify a number (two steps)

```json
{ "operation": "verifyNumber", "to": "212612345678", "brand": "Fusion" }
```

```json
{ "operation": "checkVerification", "requestId": "{{ input.request_id }}", "code": "{{ input.code }}" }
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `apiKey`, `apiSecret` and `jwt` in Fusion's **Secrets** vault. Do not commit them into version control.

- `getBalance` sends the key and secret in the URL query string: avoid logging the request URL.
- Keep the application's `private.key` out of workflows and repositories; only the generated JWT is needed.
- Each SMS, call and verification costs credit: protect workflows from loops and repeated triggers.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `JWT is required for makeCall` / `Recipient number is required...` / `Sender number is required...` / `answerUrl is required...`
- **Cause:** A field needed by `makeCall` is empty.
- **Solution:** Fill in `jwt`, `to`, `from` and `answerUrl`.

#### `Vonage API Error: 400 Bad Request` with `"name":"number"` on `makeCall`
- **Cause:** `from` or `to` is not a phone number (e.g. `Vonage APIs`).
- **Solution:** Use a phone number of up to 15 digits for both fields.

#### `Vonage API Error: 401` on `makeCall`
- **Cause:** Expired or invalid JWT.
- **Solution:** Generate a new JWT from the application ID and private key.

#### `sendSms` returns `"status": "29"` (non-whitelisted destination)
- **Cause:** Trial account sending to a number that is not a test number.
- **Solution:** Add the number under **Test numbers**, or upgrade the account.

#### `checkVerification` returns `"status": "6"`
- **Cause:** The request expired, was cancelled, or was already verified.
- **Solution:** Run `verifyNumber` again and check the new code within a few minutes.

#### `checkVerification` returns `"status": "16"`
- **Cause:** Wrong code.
- **Solution:** Ask the user to enter the code again.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-19 | Full documentation: 5 operations, API key and JWT setup, answer URL, output examples, troubleshooting |
| 1.0.0 | 2026-09-14 | Initial documentation |

<!-- /SECTION: changelog -->

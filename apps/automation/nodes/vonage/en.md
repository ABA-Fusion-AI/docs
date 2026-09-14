---
node_id: "vonage"

title: "Vonage"

description: "Send SMS messages, make voice calls, check account balance, and verify phone numbers using Vonage APIs."

category: "Communication / Messaging"

version: "1.0.0"

language: "en"

last_updated: "2026-09-14"

author: "Fusion Team"

tags:

- vonage
- nexmo
- sms
- voice
- calls
- verification
- phone
- messaging
- api

related_nodes:

- http-request
- function
- if
- infobip

---

**# Vonage**

> **\*\*Category:\*\*** communication-nodes | **\*\*Type:\*\*** Action Node

Integrate **\*\*Vonage (Nexmo)\*\*** communication APIs into Fusion workflows.

The **\*\*Vonage\*\*** node supports sending SMS messages, creating outbound voice calls, checking account balance, starting phone-number verification, and checking verification codes.

The node uses Vonage API key and secret credentials for SMS, balance, and verification operations. Voice calls use a Vonage JWT supplied through the `jwt` configuration field.

**### Supported Features**

\- Send SMS messages using the Vonage SMS API

\- Create outbound voice calls using the Vonage Voice API

\- Configure a voice-call NCCO answer URL

\- Check Vonage account balance

\- Start phone-number verification

\- Check a verification code using a request ID

\- Configure a verification brand name

\- Use Vonage API key and API secret authentication

\- Use Bearer JWT authentication for voice calls

\- Return parsed Vonage API JSON responses directly

**### Use Cases**

\- Send SMS notifications from workflows

\- Trigger automated outbound calls

\- Verify user phone numbers

\- Validate one-time verification codes

\- Check Vonage account credit

\- Build communication workflows combining SMS and voice

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"sendSms"` | `sendSms`, `makeCall`, `getBalance`, `verifyNumber`, or `checkVerification`. |
| `apiKey` | `string` | ✅ Yes | — | Vonage API key. |
| `apiSecret` | `string` | ✅ Yes | — | Vonage API secret. |
| `jwt` | `string` | ❌ No | — | Vonage JWT token for `makeCall`. |
| `from` | `string` | ❌ No | — | Sender number or name. |
| `to` | `string` | ❌ No | — | Recipient phone number. |
| `text` | `string` | ❌ No | — | SMS message text. |
| `answerUrl` | `string` | ❌ No | — | NCCO answer URL for voice calls. |
| `brand` | `string` | ❌ No | — | Brand name for verification. |
| `requestId` | `string` | ❌ No | — | Verification request ID. |
| `code` | `string` | ❌ No | — | Verification code to check. |

**### makeCall Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `jwt` | `string` | ✅ Yes | — | Bearer JWT used for Voice API authentication. |
| `to` | `string` | ✅ Yes | — | Recipient phone number. |
| `from` | `string` | ✅ Yes | — | Sender phone number. |
| `answerUrl` | `string` | ✅ Yes | — | URL returning NCCO instructions. |

**### verifyNumber Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `to` | `string` | ⚠️ Operation input | — | Number sent to Vonage as `number`. |
| `brand` | `string` | ❌ No | `"Fusion"` | Verification brand. |

**### checkVerification Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `requestId` | `string` | ⚠️ Operation input | — | Sent as `request_id`. |
| `code` | `string` | ⚠️ Operation input | — | Verification code. |

**---**

**## Operations**

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| `sendSms` | `https://rest.nexmo.com/sms/json` | `POST` | Send an SMS message. |
| `makeCall` | `https://api.nexmo.com/v1/calls` | `POST` | Create an outbound voice call. |
| `getBalance` | `https://rest.nexmo.com/account/get-balance` | `GET` | Retrieve account balance. |
| `verifyNumber` | `https://api.nexmo.com/verify/json` | `POST` | Start phone-number verification. |
| `checkVerification` | `https://api.nexmo.com/verify/check/json` | `POST` | Check a verification code. |

**---**

**## Request Body Construction**

**### Send SMS**

```json
{
  "api_key": "YOUR_API_KEY",
  "api_secret": "YOUR_API_SECRET",
  "from": "Fusion",
  "to": "212600000000",
  "text": "Hello from Fusion"
}
```

**### Make Call**

```json
{
  "to": [
    {
      "type": "phone",
      "number": "212600000000"
    }
  ],
  "from": {
    "type": "phone",
    "number": "212500000000"
  },
  "answer_url": [
    "https://example.com/answer"
  ]
}
```

Header:

```text
Authorization: Bearer <jwt>
```

**### Get Balance**

```text
GET https://rest.nexmo.com/account/get-balance?api_key=<apiKey>&api_secret=<apiSecret>
```

**### Verify Number**

```json
{
  "api_key": "YOUR_API_KEY",
  "api_secret": "YOUR_API_SECRET",
  "number": "212600000000",
  "brand": "Fusion"
}
```

**### Check Verification**

```json
{
  "api_key": "YOUR_API_KEY",
  "api_secret": "YOUR_API_SECRET",
  "request_id": "REQUEST_ID",
  "code": "1234"
}
```

**---**

**## Inputs & Outputs**

**### Inputs**

The node does not use incoming workflow data directly.

**### Outputs**

Successful requests return `response.json()` directly.

The node does not wrap or normalize successful API responses.

**---**

**## Configuration Examples**

**### Send SMS**

```json
{
  "operation": "sendSms",
  "apiKey": "YOUR_VONAGE_API_KEY",
  "apiSecret": "YOUR_VONAGE_API_SECRET",
  "from": "Fusion",
  "to": "212600000000",
  "text": "Hello from Fusion"
}
```

**### Make Call**

```json
{
  "operation": "makeCall",
  "apiKey": "YOUR_VONAGE_API_KEY",
  "apiSecret": "YOUR_VONAGE_API_SECRET",
  "jwt": "YOUR_VONAGE_JWT",
  "from": "212500000000",
  "to": "212600000000",
  "answerUrl": "https://example.com/answer"
}
```

**### Get Balance**

```json
{
  "operation": "getBalance",
  "apiKey": "YOUR_VONAGE_API_KEY",
  "apiSecret": "YOUR_VONAGE_API_SECRET"
}
```

**### Verify Number**

```json
{
  "operation": "verifyNumber",
  "apiKey": "YOUR_VONAGE_API_KEY",
  "apiSecret": "YOUR_VONAGE_API_SECRET",
  "to": "212600000000",
  "brand": "Fusion"
}
```

**### Check Verification**

```json
{
  "operation": "checkVerification",
  "apiKey": "YOUR_VONAGE_API_KEY",
  "apiSecret": "YOUR_VONAGE_API_SECRET",
  "requestId": "VERIFY_REQUEST_ID",
  "code": "1234"
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Vonage (`sendSms`)

\- Trigger → Vonage (`makeCall`)

\- Form Submission → Vonage (`verifyNumber`)

\- User Code Input → Vonage (`checkVerification`)

\- Scheduler → Vonage (`getBalance`)

\- Vonage → If

**---**

**## Error Handling**

**### Generic API Error**

```text
Vonage API Error: <status> <statusText> <response body>
```

**### Missing JWT**

```text
JWT is required for makeCall
```

**### Missing Recipient Number**

```text
Recipient number is required for makeCall
```

**### Missing Sender Number**

```text
Sender number is required for makeCall
```

**### Missing Answer URL**

```text
answerUrl is required for makeCall
```

**### Unknown Operation**

```text
Unknown operation: <operation>
```

**---**

**## Troubleshooting**

**### Voice Call Validation Errors**

The `makeCall` operation explicitly requires `jwt`, `to`, `from`, and `answerUrl`.

Provide all four values before executing the node.

---

**### SMS Request Fails**

The `sendSms` operation does not perform explicit local validation for `from`, `to`, or `text`.

Verify these values if Vonage rejects the request.

---

**### Verification Request Fails**

`verifyNumber` sends `to` directly as the API `number` field and falls back to `"Fusion"` when `brand` is missing.

---

**### Verification Check Fails**

`requestId` and `code` are passed directly to Vonage without explicit local validation.

Verify both values.

**---**

**## Security**

SMS and verification operations send:

```text
api_key
api_secret
```

in the JSON body.

Balance lookup includes credentials directly in the URL query string:

```text
?api_key=<apiKey>&api_secret=<apiSecret>
```

Voice calls use:

```text
Authorization: Bearer <jwt>
```

For production workflows:

\- Store `apiKey`, `apiSecret`, and `jwt` securely

\- Avoid logging URLs containing balance credentials

\- Avoid logging request bodies containing `api_secret`

\- Avoid exposing Bearer JWT values

\- Use trusted HTTPS `answerUrl` values

**---**

**## Notes**

The default operation is:

```text
sendSms
```

The default verification brand is:

```text
Fusion
```

The node transforms:

```text
answerUrl → answer_url
requestId → request_id
```

The node does not:

\- Generate JWTs

\- Refresh expired JWTs

\- Generate NCCO content

\- Host the answer URL

\- Poll call status

\- Poll verification status

\- Retry failed requests

\- Normalize phone numbers

\- Validate E.164 formatting locally

\- Transform successful API responses

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-14 | Initial release |

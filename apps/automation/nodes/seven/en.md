---
node_id: "seven"
title: "Seven (sms77)"
description: "Send SMS and voice calls, manage contacts, look up phone numbers and check the balance with the Seven (sms77) gateway."
category: "peer-only"
subcategory: "Communication"
version: "1.0.0"
language: "en"
last_updated: "2026-11-18"
author: "Fusion Team"
tags:
  - seven
  - sms77
  - sms
  - voice
  - messaging
  - phone-lookup
related_nodes:
  - seven-io
  - pender-sms
  - infobip-sms
  - messagebird
---

<!-- SECTION: overview -->
# Seven (sms77)

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send **SMS** and **text-to-speech voice calls**, manage **contacts** and **look up phone numbers** with the **Seven** gateway (formerly sms77, `https://gateway.seven.io/api`). The node offers **8 operations**:

| Resource | Operations |
|----------|------------|
| Messages | `sendSms`, `sendVoice`, `getVoiceStatus` |
| Contacts | `listContacts`, `createContact`, `deleteContact` |
| Utilities | `lookupNumber`, `getBalance` |

### Use Cases

- **Notifications & OTPs:** Send order updates, alerts or verification codes by SMS.
- **Voice alerts:** Call an on-call engineer with a spoken message when a critical alert fires.
- **Number validation:** Check the country, carrier and line type of a phone number before sending.
- **Contact sync:** Add the contacts of a form or a CRM to your Seven address book.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | Operation to perform (default `sendSms`). |
| `apiKey` | `string` | Yes | Seven API key. |
| `to` | `string` | Depends on operation | Recipient number in international format (e.g. `+212612345678`). Also used by `lookupNumber` and `getVoiceStatus`. |
| `text` | `string` | For `sendSms`, `sendVoice` | Message text (read aloud for voice calls). |
| `from` | `string` | No | Sender ID or phone number. |
| `flash` | `string` | No | `true` to send a flash SMS (shown on screen, not stored). |
| `contactName` | `string` | For `createContact` | Full name: first word = first name, rest = last name. |
| `number` | `string` | For `createContact` | Phone number of the contact. |
| `contactId` | `string` | For `deleteContact` | ID of the contact to delete. |
| `xml` | `string` | No | `true` if `text` is XML/SSML for `sendVoice`. |

> All parameters are shown for every operation in the builder; only the ones listed in the table below are used.

### Operations

| Operation | Endpoint | Required | Optional |
|-----------|----------|----------|----------|
| `sendSms` | `POST /sms` | `to`, `text` | `from`, `flash` |
| `sendVoice` | `POST /voice` | `to`, `text` | `from`, `xml` |
| `getVoiceStatus` | `GET /journal/voice` | — | `to` (filters the voice log by recipient) |
| `listContacts` | `GET /contacts` | — | — |
| `createContact` | `POST /contacts` | `contactName`, `number` | — |
| `deleteContact` | `DELETE /contacts/{id}` | `contactId` | — |
| `lookupNumber` | `POST /lookup` (type `format`) | `to` | — |
| `getBalance` | `GET /balance` | — | — |

> `lookupNumber` reads the number from the **`to`** field (not `number`).

### Authentication & API Key

1. Log in to **seven.io**.
2. Open **Settings → Developer → API Keys** (or *API*), and create or copy a key.
3. Paste it into `apiKey` (or store it in Fusion Secrets).

> A **sandbox** key simulates sends: the API answers `100` (success) but no SMS or call is delivered, and voice calls return a dummy ID. Use a production key to really send messages.

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
| `success` | `object` / `number` | Response of Seven. JSON answers are returned as-is; text answers are returned as `{ "result": "<text>" }`. |
| `error` | `Error` | Emitted on missing fields or HTTP errors (`Seven API error <status>: ...`). |

### Output Examples

#### `sendSms`

```json
100
```

`100` is Seven's success code (message accepted). Other codes indicate an error, e.g. `201` invalid sender, `202` invalid recipient, `500` not enough credit, `900` authentication failed.

#### `sendVoice`

```json
{ "result": "100\n123456789\n0.585" }
```

Three lines: status code, call ID, price.

#### `lookupNumber`

```json
{
  "success": true,
  "national": "06 ",
  "international": "+212600000001",
  "international_formatted": "+212 ",
  "country_name": "Morocco",
  "country_code": "212",
  "country_iso": "MA",
  "carrier": "Inwi",
  "network_type": "mobile"
}
```

#### `createContact`

```json
{
  "id": 20948294,
  "properties": {
    "firstname": "Fusion",
    "lastname": "Test",
    "mobile_number": "212600000001"
  },
  "groups": [],
  "created": "2026-11-18 01:15:51"
}
```

#### `getBalance`

```json
0.5
```

#### `deleteContact`

```json
{ "success": true, "error": null }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send an SMS with Seven
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Seven (sms77) Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Send an SMS

```json
{
  "apiKey": "{{ secrets.SEVEN_API_KEY }}",
  "operation": "sendSms",
  "to": "+212612345678",
  "text": "Your order {{ input.orderId }} has shipped.",
  "from": "Fusion"
}
```

#### Voice alert

```json
{
  "apiKey": "{{ secrets.SEVEN_API_KEY }}",
  "operation": "sendVoice",
  "to": "+212612345678",
  "text": "Critical alert: the payment service is down."
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Seven `apiKey` in Fusion's **Secrets** vault. Do not commit it into version control.

- Each SMS and voice call costs credit: protect workflows that send messages from loops and repeated triggers.
- Respect local rules on commercial messages and phone calls (consent, opt-out).

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `to is required for sendSms` / `text is required for sendSms` (same for `sendVoice`)
- **Cause:** The recipient or the text is empty.
- **Solution:** Fill in `to` and `text`.

#### `to (number) is required for lookupNumber`
- **Cause:** The number was put in `number` instead of `to`.
- **Solution:** Put the number to look up in `to`.

#### `contactName is required...` / `number is required for createContact` / `contactId is required for deleteContact`
- **Cause:** A contact field is empty.
- **Solution:** Fill in the field (get the contact ID with `listContacts`).

#### `Seven API error 401` / result `900`
- **Cause:** Invalid API key.
- **Solution:** Copy the key again from **Settings → Developer**.

#### `sendSms` returns `100` but nothing is received
- **Cause:** Sandbox API key, or the recipient blocks short codes / unknown senders.
- **Solution:** Use a production key and check the journal on seven.io.

#### `sendSms` returns `500`
- **Cause:** Not enough credit.
- **Solution:** Top up the account (check with `getBalance`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-18 | Full documentation: 8 operations, parameters, sandbox note, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->

---
node_id: "messagebird"
title: "MessageBird"
description: "Send SMS and voice calls, look up phone numbers and read messages with the MessageBird (Bird) platform API."
category: "peer-only"
subcategory: "Communication"
version: "1.0.0"
language: "en"
last_updated: "2026-11-19"
author: "Fusion Team"
tags:
  - messagebird
  - bird
  - sms
  - voice
  - messaging
  - phone-lookup
related_nodes:
  - seven
  - infobip-sms
  - pender-sms
---

<!-- SECTION: overview -->
# MessageBird

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send **SMS** and **voice calls**, **look up phone numbers** and read your **SMS history** with **MessageBird**, now called **Bird** (`https://<region>.platform.bird.com/v1`). The node offers **6 operations**:

| Resource | Operations |
|----------|------------|
| Messages | `sendSms`, `listMessages`, `getMessageById` |
| Voice | `sendVoiceMessage` |
| Utilities | `lookupNumber`, `getWorkspace` |

### Use Cases

- **Notifications & OTPs:** Send order updates, alerts or verification codes by SMS.
- **Voice alerts:** Call an on-call engineer with a voice sequence when a critical alert fires.
- **Number validation:** Check a phone number before sending to it.
- **Delivery tracking:** List sent messages or read the status of one message.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | Operation to perform (default `sendSms`). |
| `apiKey` | `string` | Yes | Bird API key (`bk_<region>_...`). The region in the key selects the API host (e.g. `bk_eu1_...` → `https://eu1.platform.bird.com`). |
| `originator` | `string` | For `sendSms`, `sendVoiceMessage` | Sender. For SMS: a sender configured in your workspace. For voice: a phone number in international format (e.g. `+14155550100`). |
| `recipients` | `string` | For `sendSms`, `sendVoiceMessage` | Recipient numbers, comma-separated (e.g. `+212612345678,+212698765432`). One message or call is sent per recipient. |
| `body` | `string` | For `sendSms` | SMS text. |
| `category` | `string` | No | SMS category: `transactional` (default) or `authentication`. |
| `phoneNumber` | `string` | For `lookupNumber` | Number to look up, in international format. |
| `messageId` | `string` | For `getMessageById` | ID of an SMS message. |
| `voiceSequenceId` | `string` | For `sendVoiceMessage` | ID of a voice sequence created in the Bird dashboard (`vsq_...`). |
| `voiceEntryNodeId` | `string` | For `sendVoiceMessage` | Step ID of the sequence's entry step (e.g. `start`). |
| `limit` | `number` | No | Number of messages returned by `listMessages` (default `20`). |

> All parameters are shown for every operation in the builder; only the ones listed in the table below are used.

### Operations

| Operation | Endpoint | Required | Optional |
|-----------|----------|----------|----------|
| `sendSms` | `POST /sms/messages` | `originator`, `recipients`, `body` | `category` |
| `sendVoiceMessage` | `POST /voice/calls` | `originator`, `recipients`, `voiceSequenceId`, `voiceEntryNodeId` | — |
| `getWorkspace` | `GET /workspace` | — | — |
| `lookupNumber` | `POST /lookup/phone-number` | `phoneNumber` | — |
| `listMessages` | `GET /sms/messages?limit=` | — | `limit` |
| `getMessageById` | `GET /sms/messages/{id}` | `messageId` | — |

### Authentication & API Key

1. Log in to the **Bird** dashboard (`bird.com`).
2. Open the **API keys** settings of your workspace and create a key.
3. Paste it into `apiKey` (or store it in Fusion Secrets).

### Workspace setup (before sending)

- **SMS senders:** Add the sender you will use as `originator` under **SMS senders**, and register it for destination countries that require it.
- **SMS destinations:** Enable each destination country (e.g. Morocco) in the **SMS destination settings**. A change takes up to a minute to apply.
- **Voice sequence:** Create a voice sequence (e.g. *Start call* → *Say*), publish it, and copy its ID (`vsq_...`) and the **Step ID** of its first step (shown in the step's **Details** tab, e.g. `start`).
- **Wallet:** `lookupNumber`, SMS and calls are paid from the organization wallet.

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
| `success` | `object` | Response of Bird. With several recipients, `sendSms` returns `{ "messages": [...] }` and `sendVoiceMessage` returns `{ "calls": [...] }`. |
| `error` | `Error` | Emitted on missing fields or HTTP errors (`MessageBird API error (<status>): {...}`). |

### Output Examples

#### `getWorkspace`

```json
{
  "id": "ws_01...",
  "name": "fusiontest",
  "organization_id": "org_01...",
  "notification_emails": { "operational": [] },
  "suspended_at": null,
  "created_at": "2026-10-07T11:23:18.545538Z",
  "updated_at": "2026-10-07T11:23:18.545538Z"
}
```

#### `listMessages`

```json
{
  "data": [],
  "next_cursor": null,
  "prev_cursor": null,
  "refresh_cursor": null
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use MessageBird in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **MessageBird Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Send an SMS

```json
{
  "apiKey": "{{ secrets.BIRD_API_KEY }}",
  "operation": "sendSms",
  "originator": "Fusion",
  "recipients": "+212612345678",
  "body": "Your order {{ input.orderId }} has shipped."
}
```

#### Voice alert

```json
{
  "apiKey": "{{ secrets.BIRD_API_KEY }}",
  "operation": "sendVoiceMessage",
  "originator": "+14155550100",
  "recipients": "+212612345678",
  "voiceSequenceId": "vsq_...",
  "voiceEntryNodeId": "start"
}
```

#### Look up a number

```json
{
  "apiKey": "{{ secrets.BIRD_API_KEY }}",
  "operation": "lookupNumber",
  "phoneNumber": "+212612345678"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Bird `apiKey` in Fusion's **Secrets** vault. Do not commit it into version control.

- Each SMS, call and lookup costs credit: protect workflows that send messages from loops and repeated triggers.
- Respect local rules on commercial messages and phone calls (consent, opt-out).

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `originator is required` / `recipients is required` / `body is required`
- **Cause:** A required field is empty.
- **Solution:** Fill in the sender, the recipients and the text.

#### `voiceSequenceId is required`
- **Cause:** No voice sequence ID.
- **Solution:** Create a voice sequence in the Bird dashboard and paste its `vsq_...` ID.

#### `MessageBird API error (401)`
- **Cause:** Invalid or revoked API key.
- **Solution:** Create a new key in the Bird dashboard.

#### `MessageBird API error (422)`: `SMSSenderNotConfigured`
- **Cause:** The `originator` is not one of the workspace's SMS senders.
- **Solution:** Add it under **SMS senders**, or use a sender that is already configured.

#### `MessageBird API error (422)`: `SMSDestinationNotEnabled`
- **Cause:** The destination country is not enabled.
- **Solution:** Enable it in the **SMS destination settings**, wait a minute and retry.

#### `MessageBird API error (422)` on `sendVoiceMessage` (`from` pattern / `entry_node_id`)
- **Cause:** `originator` is not a phone number, or `voiceEntryNodeId` is empty.
- **Solution:** Use a calling number in international format and the Step ID of the sequence's first step.

#### `MessageBird API error (402)`: `LookupInsufficientBalance`
- **Cause:** The organization wallet is empty.
- **Solution:** Top up the wallet in **Billing**, then retry.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-19 | Full documentation: 6 operations, parameters, workspace setup, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->

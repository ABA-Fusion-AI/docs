---
node_id: infobip-sms-email
title: Infobip SMS/Email
description: Send SMS or Email messages using the Infobip API.
category: Communication
version: 1.0.0
language: TypeScript
last_updated: 2026-09-04
author: ABA Fusion AI
tags:
  - infobip
  - sms
  - email
  - messaging
  - communication
related_nodes: []
---

# Infobip SMS/Email

**Category:** Communication  
**Type:** Action Node

## Overview

The **Infobip SMS/Email** node sends SMS or Email messages using the Infobip API.

The node supports two channels:

- `sms`
- `email`

For SMS messages, the destination phone number is automatically normalized to international format. If a Moroccan number is provided without a country code, the node automatically applies the `+212` prefix.

The node also includes automatic retry handling for temporary Infobip API errors such as rate limits and server errors.

## Operations

### Send SMS

Sends a text message through the Infobip SMS API.

**Endpoint:**

```text
POST /sms/2/text/advanced
```

Required fields:

- `apiKey`
- `baseUrl`
- `channel`
- `sender`
- `to`
- `body`

Set:

```text
channel = sms
```

### Send Email

Sends a plain-text email through the Infobip Email API.

**Endpoint:**

```text
POST /email/3/send
```

Required fields:

- `apiKey`
- `baseUrl`
- `channel`
- `sender`
- `to`
- `subject`
- `body`

Set:

```text
channel = email
```

## Parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `apiKey` | string | Yes | — | Infobip API key used for authentication. |
| `baseUrl` | URL | No | `https://api.infobip.com` | Infobip API base URL. |
| `channel` | enum | No | `sms` | Message channel. Supported values are `sms` and `email`. |
| `sender` | string | Conditional | — | SMS sender or Email sender address. Required for both SMS and Email. |
| `to` | string | Yes | — | Destination phone number for SMS or recipient email address for Email. |
| `body` | string | Yes | — | SMS text or Email body. |
| `subject` | string | Conditional | — | Email subject. Required when `channel` is `email`. |

## SMS Phone Number Normalization

For SMS messages, the node normalizes the destination number before sending it.

Examples:

```text
0612345678
```

becomes:

```text
+212612345678
```

```text
612345678
```

becomes:

```text
+212612345678
```

```text
+212612345678
```

remains:

```text
+212612345678
```

Spaces, dashes, and parentheses are removed automatically.

## Outputs

### Successful SMS Response

Example:

```json
{
  "success": true,
  "messageId": "abc123",
  "phone": "+212612345678",
  "sender": "InfoSMS",
  "body": "Hello from Infobip",
  "response": {
    "bulkId": "bulk-id",
    "messages": [
      {
        "messageId": "abc123"
      }
    ]
  }
}
```

### Successful Email Response

Example:

```json
{
  "success": true,
  "messageId": "fna47ax9yhx83ha1ddnr",
  "to": "recipient@example.com",
  "from": "sender@example.com",
  "subject": "Test Email",
  "body": "Hello from Infobip",
  "response": {
    "bulkId": "2z7g3dxzmt6o5xfj156g",
    "messages": [
      {
        "to": "recipient@example.com",
        "messageId": "fna47ax9yhx83ha1ddnr",
        "status": {
          "groupId": 1,
          "groupName": "PENDING",
          "id": 26,
          "name": "PENDING_ACCEPTED",
          "description": "Message accepted, pending for delivery."
        }
      }
    ]
  }
}
```

### Error Response

Example:

```json
{
  "success": false,
  "error": "Infobip API error: Unauthorized",
  "channel": "sms"
}
```

## Examples

### Send an SMS

Configuration:

```json
{
  "apiKey": "YOUR_INFOBIP_API_KEY",
  "baseUrl": "https://api.infobip.com",
  "channel": "sms",
  "sender": "InfoSMS",
  "to": "0612345678",
  "body": "Hello from Infobip"
}
```

The node normalizes the phone number and sends the request to:

```text
https://api.infobip.com/sms/2/text/advanced
```

### Send an Email

Configuration:

```json
{
  "apiKey": "YOUR_INFOBIP_API_KEY",
  "baseUrl": "https://api.infobip.com",
  "channel": "email",
  "sender": "sender@example.com",
  "to": "recipient@example.com",
  "subject": "Test Email",
  "body": "Hello from Infobip"
}
```

The node sends the request to:

```text
https://api.infobip.com/email/3/send
```

## cURL Tests

### Test SMS

```bash
curl.exe -X POST "https://api.infobip.com/sms/2/text/advanced" ^
  -H "Authorization: App YOUR_INFOBIP_API_KEY" ^
  -H "Content-Type: application/json" ^
  -H "User-Agent: infobip-workflow-node" ^
  -d "{\"messages\":[{\"destinations\":[{\"to\":\"+212612345678\"}],\"from\":\"InfoSMS\",\"text\":\"Hello from Infobip\"}]}"
```

### Test Email

```bash
curl.exe -X POST "https://api.infobip.com/email/3/send" ^
  -H "Authorization: App YOUR_INFOBIP_API_KEY" ^
  -H "User-Agent: infobip-workflow-node" ^
  -F "from=sender@example.com" ^
  -F "to=recipient@example.com" ^
  -F "subject=Test Email" ^
  -F "text=Hello from Infobip"
```

## Retry Behavior

The node automatically retries requests when Infobip returns one of the following HTTP status codes:

```text
429
500
502
503
504
```

The node uses exponential backoff between retry attempts.

Default configuration:

```text
Maximum retries: 4
Backoff factor: 2
```

The delays are approximately:

```text
1 second
2 seconds
4 seconds
8 seconds
```

## Troubleshooting

### Sender must be specified for SMS messages

**Error:**

```text
Sender must be specified for SMS messages
```

**Cause:**

The `sender` field is empty while `channel` is set to `sms`.

**Solution:**

Provide an SMS sender value.

---

### Destination must be specified for SMS messages

**Error:**

```text
Destination must be specified for SMS messages
```

**Cause:**

The `to` field is empty.

**Solution:**

Provide the destination phone number.

---

### Subject must be specified for email messages

**Error:**

```text
Subject must be specified for email messages
```

**Cause:**

The `subject` field is empty while `channel` is set to `email`.

**Solution:**

Provide an email subject.

---

### Sender must be specified for email messages

**Error:**

```text
Sender (from) must be specified for email messages
```

**Cause:**

The `sender` field is empty while using the Email channel.

**Solution:**

Provide the sender email address.

---

### Infobip API error

**Error:**

```text
Infobip API error: ...
```

**Cause:**

Infobip rejected the request.

Possible causes include:

- Invalid API key
- Invalid sender
- Invalid destination
- Invalid email sender
- Insufficient account permissions
- Infobip account or channel configuration issues

**Solution:**

Check the Infobip API response returned by the node and verify the configured credentials and message parameters.

---

### Email status is PENDING_ACCEPTED

Example:

```json
{
  "groupName": "PENDING",
  "name": "PENDING_ACCEPTED",
  "description": "Message accepted, pending for delivery."
}
```

This means Infobip successfully accepted the message and it is waiting to be processed for delivery.

The response also contains the `messageId`, which can be used to identify the message.

## Version History

### 1.0.0

- Added SMS support through Infobip SMS API.
- Added Email support through Infobip Email API.
- Added automatic Moroccan phone number normalization.
- Added retry handling for temporary API errors.
- Added message ID and Infobip API response to node outputs.

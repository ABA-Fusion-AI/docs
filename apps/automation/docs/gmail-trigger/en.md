---
node_id: "gmail-trigger"
title: "Gmail - New Message Trigger"
description: "Poll Gmail and emit newly detected messages"
category: "triggers"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - gmail
  - email
  - google
  - polling
related_nodes:
  - gmail
---

<!-- SECTION: header -->

# Gmail - New Message Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls Gmail on an interval and emits messages not seen before.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Gmail - New Message Trigger** node uses OAuth2 credentials to periodically list messages, tracks IDs in memory, and emits only newly discovered ones.

### Key Features

- Polling-based new message trigger
- Optional label and query filtering
- Warmup behavior to avoid emitting existing emails on first poll
- Metadata fallback on restricted scopes
- Error output emission for authentication/query scope issues

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter      | Type       | Required | Default | Description                   |
| -------------- | ---------- | -------- | ------- | ----------------------------- |
| `clientId`     | `string`   | ✅ Yes   | —       | Google OAuth client ID        |
| `clientSecret` | `string`   | ✅ Yes   | —       | Google OAuth client secret    |
| `refreshToken` | `string`   | ✅ Yes   | —       | OAuth refresh token           |
| `userId`       | `string`   | ❌ No    | `me`    | Gmail user ID                 |
| `labelIds`     | `string[]` | ❌ No    | —       | Label filters for list API    |
| `q`            | `string`   | ❌ No    | —       | Gmail search query            |
| `intervalMs`   | `number`   | ❌ No    | `15000` | Poll interval in milliseconds |

### Trigger Behavior

1. First poll runs in warmup mode and stores current message IDs.
2. Scheduled polls list messages (up to 50 each run).
3. New IDs are fetched in detail (`full`, with metadata fallback if needed).
4. Emits an array of new messages.
5. Emits to `error` output for known auth/scope errors.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

This is a trigger node and does not accept upstream input.

### Outputs

| Output   | Type     | Description                        |
| -------- | -------- | ---------------------------------- |
| `output` | `array`  | Array of new Gmail message objects |
| `error`  | `object` | Error payload: `{ error, reason }` |

### Output Example

```json
[
  {
    "id": "18f3f3abc123",
    "threadId": "18f3f3abc000",
    "snippet": "Hello from Fusion...",
    "body": "Hello from Fusion..."
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Trigger on new inbox emails

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "labelIds": ["INBOX"],
  "intervalMs": 15000
}
```

### Trigger unread support emails only

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "q": "to:support@example.com is:unread",
  "intervalMs": 30000
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Trigger stops with `invalid_grant`

The refresh token is invalid/revoked. Re-authorize and update credentials.

#### `q` parameter rejected with metadata scope

OAuth scopes do not allow Gmail query filtering in metadata mode. Use broader scopes or remove `q`.

#### No messages emitted on start

Expected behavior: initial poll is warmup and does not emit existing emails.

<!-- /SECTION: troubleshooting -->

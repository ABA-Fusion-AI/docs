---
node_id: "gmail-trigger"
title: "Gmail - New Message Trigger"
description: "Poll Gmail and emit newly detected messages"
category: "triggers"
subcategory: "google"
version: "1.1.0"
language: "en"
last_updated: "2026-07-16"
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

| Parameter      | Type       | Required | Default                     | Description                                                        |
| -------------- | ---------- | -------- | --------------------------- | ------------------------------------------------------------------ |
| `clientId`     | `string`   | ✅ Yes   | —                           | Google OAuth client ID. Required — see *Why the credentials*        |
| `clientSecret` | `string`   | ✅ Yes   | —                           | Google OAuth client secret. Required                                |
| `redirectUri`  | `url`      | ❌ No    | `urn:ietf:wg:oauth:2.0:oob` | OAuth redirect URI. Leave empty for the out-of-band default         |
| `refreshToken` | `string`   | ✅ Yes   | —                           | OAuth refresh token identifying the mailbox to poll                 |
| `userId`       | `string`   | ❌ No    | `me`                        | Mailbox to poll. `me` is the mailbox the refresh token belongs to   |
| `labelIds`     | `string[]` | ❌ No    | —                           | Only emit messages carrying these label **IDs**, e.g. `INBOX`       |
| `q`            | `string`   | ❌ No    | —                           | Gmail search query. Not supported under metadata-only scopes        |
| `intervalMs`   | `number`   | ❌ No    | `15000`                     | Poll interval in milliseconds                                       |

### Why the credentials are required

A refresh token alone cannot authenticate. On every **poll** the node exchanges the
refresh token for a short-lived access token, and Google's token endpoint requires
`client_id` and `client_secret` in that exchange — so all three are mandatory.

`redirectUri` is only needed if your OAuth client requires a specific one. Left
empty, the trigger uses the out-of-band default (`urn:ietf:wg:oauth:2.0:oob`),
which is not itself a URL and so cannot be typed into the field.

### Finding label IDs

`labelIds` filters on label **IDs** (e.g. `Label_123`), not names. Run the
[Gmail](../gmail/en.md) node's `listLabels` operation to get every label's `id` and
`name`. System labels use fixed IDs such as `INBOX`, `UNREAD`, and `STARRED`.

### Choosing a poll interval

Each poll costs one API call, plus one more per new message discovered, against the
mailbox's quota. `intervalMs` is not lower-bounded, so a small value will burn quota
quickly — the 15s default is a reasonable floor for most mailboxes.

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

| Output    | Type     | Description                                    |
| --------- | -------- | ---------------------------------------------- |
| `success` | `array`  | Array of newly discovered, parsed Gmail messages |
| `error`   | `object` | Error payload: `{ error, reason }`             |

### Output Example

Each message is parsed into the **same shape the [Gmail](../gmail/en.md) node's
`get` operation returns** — the trigger runs the same parser — so a downstream
node can treat both sources identically. Attachment bodies are fetched and filled
into `contentBase64` automatically; under the metadata-scope fallback
`gmail.reducedFormat` is `true` and attachment content is not fetched.

```jsonc
[
  {
    "subject": "…",
    "from": { "name": "…", "email": "…", "raw": "…" },
    "recipients": {
      "to": [{ "name": "…", "email": "…", "raw": "…" }],
      "cc": [],
      "bcc": [],
      "replyTo": []
    },
    "message": { "text": "…", "html": "…", "preview": "…" },
    "attachments": [
      {
        "fileName": "invoice.pdf",
        "mimeType": "application/pdf",
        "sizeBytes": 12345,
        "encoding": "base64",
        "contentBase64": "JVBERi0xLjQK…",
        "isInline": false
      }
    ],
    "gmail": {
      "messageId": "18f3f3abc123",
      "threadId": "18f3f3abc000",
      "labels": ["INBOX"],
      "date": "…",
      "internalDate": "…",
      "rfc822MessageId": "…",
      "reducedFormat": false
    }
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

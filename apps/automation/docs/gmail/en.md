---
node_id: "gmail"
title: "Gmail"
description: "Send and manage Gmail messages and labels"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - gmail
  - email
  - google
related_nodes:
  - gmail-trigger
---

<!-- SECTION: header -->

# Gmail

> **Category:** Integrations | **Type:** Action Node

Run Gmail operations: `send`, `get`, `delete`, `move`, `markUnread`, and `createLabel`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Gmail** node authenticates with OAuth2 refresh token credentials and executes message or label operations on the authenticated mailbox.

### Key Features

- Send HTML/text emails with optional attachments
- Fetch message list with parsed fields
- Delete and move messages via label modifications
- Create labels
- Metadata-scope fallback for restricted Gmail scopes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter          | Type       | Required | Default                     | Description                                                          |
| ------------------ | ---------- | -------- | --------------------------- | -------------------------------------------------------------------- |
| `clientId`         | `string`   | ✅ Yes   | —                           | Google OAuth client ID                                               |
| `clientSecret`     | `string`   | ✅ Yes   | —                           | Google OAuth client secret                                           |
| `redirectUri`      | `string`   | ❌ No    | `urn:ietf:wg:oauth:2.0:oob` | OAuth redirect URI                                                   |
| `refreshToken`     | `string`   | ✅ Yes   | —                           | OAuth refresh token                                                  |
| `operation`        | `enum`     | ❌ No    | `get`                       | One of: `send`, `get`, `delete`, `move`, `markUnread`, `createLabel` |
| `to`               | `string`   | ✅ Yes\* | —                           | Recipient email(s) for `send`                                        |
| `from`             | `string`   | ❌ No    | `me`                        | Sender used in MIME headers for `send`                               |
| `subject`          | `string`   | ❌ No    | `""`                        | Subject for `send`                                                   |
| `text`             | `string`   | ❌ No    | —                           | Plain text body for `send`                                           |
| `html`             | `string`   | ❌ No    | —                           | HTML body for `send`                                                 |
| `attachments`      | `array`    | ❌ No    | —                           | Attachments for `send`: `{ filename, path? , content? }`             |
| `query`            | `string`   | ❌ No    | —                           | Gmail search query for `get`                                         |
| `maxResults`       | `number`   | ❌ No    | —                           | Max listed messages for `get`                                        |
| `emailId`          | `string`   | ✅ Yes\* | —                           | Required for `delete`, `move`, `markUnread`                          |
| `labelIdsToAdd`    | `string[]` | ❌ No    | —                           | Labels to add in `move`                                              |
| `labelIdsToRemove` | `string[]` | ❌ No    | —                           | Labels to remove in `move`                                           |
| `labelName`        | `string`   | ✅ Yes\* | —                           | Label name for `createLabel`                                         |

\* Required only for the listed operations.

### Operation Behavior

| Operation     | Behavior                                                          | Output                          |
| ------------- | ----------------------------------------------------------------- | ------------------------------- |
| `send`        | Builds MIME message and calls `users.messages.send`               | Gmail message send response     |
| `get`         | Lists messages then fetches each message (full/metadata fallback) | `{ messages: ParsedMessage[] }` |
| `delete`      | Deletes one message by `emailId`                                  | Gmail API response              |
| `move`        | Modifies labels on one message                                    | Updated message metadata        |
| `markUnread`  | Removes `UNREAD` label from message by `emailId`                  | Updated message metadata        |
| `createLabel` | Creates label by name                                             | Created label object            |

### Parsed Message Shape (`get`)

Each parsed message includes fields such as:

- `id`, `threadId`, `subject`, `snippet`, `date`
- `from`, `to`, `headers`
- `textBody`, `htmlBody`
- `attachments[]` (filename, mimeType, size, attachmentId)
- `__reduced_format` (true when metadata fallback was used)

<!-- /SECTION: configuration -->

---

<!-- SECTION: examples -->

## Examples

### Send email with HTML body

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "operation": "send",
  "to": "user@example.com",
  "subject": "Welcome",
  "html": "<h1>Hello</h1><p>Welcome to Fusion.</p>"
}
```

### Get latest messages using query

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "operation": "get",
  "query": "in:inbox is:unread",
  "maxResults": 20
}
```

### Move a message to a label

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "operation": "move",
  "emailId": "18f3f3abc123",
  "labelIdsToAdd": ["Label_123"],
  "labelIdsToRemove": ["INBOX"]
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### `invalid_grant` or auth failures

Refresh token may be revoked/expired, or OAuth app credentials are wrong.

#### Metadata scope limitations

When full message scope is unavailable, node falls back to metadata and sets `__reduced_format=true`.

#### Attachment not sent

For attachments, provide either `path` (read from disk) or `content` string with a `filename`.

<!-- /SECTION: troubleshooting -->

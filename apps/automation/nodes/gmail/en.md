---
node_id: "gmail"
title: "Gmail"
description: "Send and manage Gmail messages and labels"
category: "integrations"
subcategory: "google"
version: "2.0.0"
language: "en"
last_updated: "2026-07-16"
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

Run Gmail operations: `send`, `get`, `trash`, `delete`, `move`, `markRead`, `markUnread`, `createLabel`, and `listLabels`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Gmail** node authenticates with OAuth2 refresh token credentials and executes message or label operations on the authenticated mailbox.

### Key Features

- Send HTML or plain-text emails with optional binary attachments
- Search messages and return parsed fields, including attachment content
- Trash (reversible) or permanently delete messages
- Move/archive messages via label modifications; mark read or unread
- Create labels and list existing label IDs
- Metadata-scope fallback for restricted Gmail scopes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter          | Type       | Required | Default                     | Description                                                          |
| ------------------ | ---------- | -------- | --------------------------- | -------------------------------------------------------------------- |
| `clientId`         | `string`   | ✅ Yes   | —                           | Google OAuth client ID. Required — see *Why the client ID and secret* |
| `clientSecret`     | `string`   | ✅ Yes   | —                           | Google OAuth client secret. Required                                  |
| `redirectUri`      | `url`      | ❌ No    | `urn:ietf:wg:oauth:2.0:oob` | OAuth redirect URI. Leave empty for the out-of-band default           |
| `refreshToken`     | `string`   | ✅ Yes   | —                           | OAuth refresh token identifying the mailbox to act on                 |
| `operation`        | `enum`     | ❌ No    | `get`                       | See *Operation Behavior* below                                        |
| `to`               | `email[]`  | ✅ Yes\* | —                           | Recipients for `send`. One entry per address, at least one            |
| `cc`               | `email[]`  | ❌ No    | —                           | Cc recipients for `send`. One entry per address                       |
| `bcc`              | `email[]`  | ❌ No    | —                           | Bcc recipients for `send`. One entry per address                      |
| `from`             | `string`   | ❌ No    | `me`                        | Sender for `send`. Must be an alias the mailbox may send as           |
| `subject`          | `string`   | ❌ No    | `""`                        | Subject for `send`                                                    |
| `bodyType`         | `enum`     | ❌ No    | `html`                      | `html` or `text` — which body to send. Only the selected one is sent  |
| `html`             | `string`   | ❌ No    | —                           | HTML body. Used when `bodyType=html`                                  |
| `text`             | `string`   | ❌ No    | —                           | Plain text body. Used when `bodyType=text`                            |
| `attachments`      | `array`    | ❌ No    | —                           | Attachments for `send`: `{ fileName, mimeType?, contentBase64 }`      |
| `query`            | `string`   | ❌ No    | —                           | Gmail search query for `get`                                          |
| `maxResults`       | `number`   | ❌ No    | —                           | Max listed messages for `get` (1-500)                                 |
| `emailId`          | `string`   | ✅ Yes\* | —                           | Required for `trash`, `delete`, `move`, `markRead`, `markUnread`      |
| `labelIdsToAdd`    | `string[]` | ❌ No†   | —                           | Label IDs to add in `move`. Find IDs with `listLabels`                |
| `labelIdsToRemove` | `string[]` | ❌ No†   | —                           | Label IDs to remove in `move`. Find IDs with `listLabels`             |
| `labelName`        | `string`   | ✅ Yes\* | —                           | Label name for `createLabel`                                          |

\* Required only for the listed operations. Fields hidden by the current
`operation` are never required — the dependency rule suppresses the check — so
`to` is mandatory for `send` without blocking `get` or `listLabels`.

† Each label list is optional on its own, but `move` needs **at least one of the
two** and fails with an error if both are empty. Gmail's `modify` accepts a
request with neither and reports success while changing nothing, so the node
rejects it rather than let a no-op look like a successful move.

### Recipients

`to`, `cc`, and `bcc` are **arrays of email addresses**, one entry per recipient,
each validated individually. `to` requires at least one entry.

`from` is a single string rather than an email field, because it accepts a display
name — `Support <support@example.com>` — which a strict email input rejects.

### Choosing the body format

`bodyType` selects which body is sent, and **only that one is included** — setting
`bodyType=html` ignores `text` entirely, and vice versa. The builder only ever
produces a single body part, so the selector makes explicit what the node was
previously deciding for you (it silently preferred `html`).

An empty body is allowed — useful for attachment-only emails.

### Why the client ID and secret are required

A refresh token alone cannot authenticate. On every run the node exchanges the
refresh token for a short-lived access token, and Google's token endpoint requires
`client_id` and `client_secret` in that exchange. Without them the request is
rejected, so all three credentials are mandatory.

### Attachments (`send`)

Each attachment carries its own bytes as a base64 string — nothing is read from disk:

| Field            | Type     | Required | Description                                                                     |
| ---------------- | -------- | -------- | ------------------------------------------------------------------------------- |
| `fileName`       | `string` | ✅ Yes   | File name shown to the recipient, e.g. `invoice.pdf`                             |
| `mimeType`       | `enum`   | ❌ No    | Content type, picked from a list. Defaults to `application/octet-stream`         |
| `customMimeType` | `string` | ✅ Yes\* | Free-text content type. Shown only when `mimeType` is `custom`                    |
| `contentBase64`  | `string` | ✅ Yes   | File contents as base64. A `data:` URI is also accepted                          |

\* Required only when `mimeType` is `custom`.

`mimeType` is a dropdown of common types (PDF, PNG, JPEG, CSV, ZIP, the Office
formats, …). Anything not listed is covered by the `custom` option, which reveals a
`customMimeType` field. Each attachment row toggles independently.

`customMimeType` is also how you pass a MIME type through from an upstream node:
set `mimeType` to `custom` and bind `customMimeType` to the expression.

It is required whenever it is visible: picking `custom` and leaving it empty fails
validation on `attachments.<n>.customMimeType`. While `mimeType` is anything else
the field is hidden and its value is ignored, so it never blocks the other options.

Invalid base64 fails the node with an error naming the file rather than sending a
corrupt attachment.

#### Forwarding an attachment from a `get`

`fileName` and `contentBase64` line up with what `get` returns, so bind them to its
output directly. `mimeType` is a dropdown, so pass the upstream type through the
`custom` option instead:

```jsonc
{
  "fileName": "{{ $node.get.messages[0].attachments[0].fileName }}",
  "mimeType": "custom",
  "customMimeType": "{{ $node.get.messages[0].attachments[0].mimeType }}",
  "contentBase64": "{{ $node.get.messages[0].attachments[0].contentBase64 }}"
}
```

> **Binding the whole `attachments` array** to a single expression (e.g.
> `{{ $node.get.messages[0].attachments }}`) only works if every forwarded file's
> type is one of the listed options. Config is validated *after* expressions are
> resolved, so an unlisted type such as `image/heic` fails the run with
> `invalid_enum_value`. Bind per attachment, as above, when the types are unknown
> ahead of time.

### Operation Behavior

| Operation     | Behavior                                                                       | Output                          |
| ------------- | ------------------------------------------------------------------------------ | ------------------------------- |
| `send`        | Builds a MIME message and calls `users.messages.send`                          | Gmail message send response     |
| `get`         | Searches, then fetches each message (full/metadata fallback)                   | `{ messages: ParsedMessage[] }` |
| `trash`       | Moves one message to Trash. **Reversible** — recoverable for 30 days           | Gmail API response              |
| `delete`      | ⚠️ **Permanently** deletes one message, bypassing Trash. **Cannot be undone**   | Gmail API response              |
| `move`        | Adds/removes labels on one message (remove `INBOX` to archive)                 | Updated message metadata        |
| `markRead`    | Marks a message read (removes the `UNREAD` label)                              | Updated message metadata        |
| `markUnread`  | Marks a message unread (adds the `UNREAD` label)                               | Updated message metadata        |
| `createLabel` | Creates a label by name                                                        | Created label object            |
| `listLabels`  | Lists all labels with their IDs                                                | `{ labels: [{ id, name, type }] }` |

> **Prefer `trash` over `delete`.** `delete` calls `users.messages.delete`, which
> permanently removes the message with no recovery path — Google's own guidance is
> to use Trash unless you specifically need permanent deletion.

Gmail models read state as the presence of the `UNREAD` label, so `markRead`
removes it and `markUnread` adds it.

### Finding label IDs

`move` takes label **IDs** (e.g. `Label_123`), not names. Run the `listLabels`
operation to get every label's `id` and `name`. System labels use fixed IDs such as
`INBOX`, `UNREAD`, `STARRED`, and `SPAM`.

### Parsed Message Shape (`get`)

Each parsed message has the shape:

```jsonc
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
    "messageId": "…",
    "threadId": "…",
    "labels": ["INBOX"],
    "date": "…",
    "internalDate": "…",
    "rfc822MessageId": "…",
    "reducedFormat": false
  }
}
```

Attachment bodies are fetched and filled into `contentBase64` automatically. When
the metadata-scope fallback is used, `gmail.reducedFormat` is `true` and attachment
content is not fetched.

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
  "to": ["user@example.com"],
  "subject": "Welcome",
  "bodyType": "html",
  "html": "<h1>Hello</h1><p>Welcome to Fusion.</p>"
}
```

### Send an email with an attachment

`contentBase64` is an expression field, so it can reference an upstream node's
output — here, forwarding an attachment straight from a Gmail `get` node:

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "operation": "send",
  "to": ["user@example.com", "billing@example.com"],
  "cc": ["manager@example.com"],
  "subject": "Your invoice",
  "bodyType": "html",
  "html": "<p>Invoice attached.</p>",
  "attachments": [
    {
      "fileName": "invoice.pdf",
      "mimeType": "application/pdf",
      "contentBase64": "JVBERi0xLjQKJcfsj6IKNSAwIG9iago8PC9MZW5ndGg..."
    }
  ]
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

### Archive a message under a label

Removing `INBOX` archives the message. Get `Label_123` from `listLabels`.

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

### Trash a message

Prefer this over `delete` — the message stays recoverable for 30 days. Credentials
are omitted below for brevity; all three are still required.

```json
{
  "operation": "trash",
  "emailId": "18f3f3abc123"
}
```

### Permanently delete a message

⚠️ Bypasses Trash. There is no recovery path.

```json
{
  "operation": "delete",
  "emailId": "18f3f3abc123"
}
```

### Mark a message read or unread

`markRead` removes the `UNREAD` label; `markUnread` adds it.

```json
{
  "operation": "markRead",
  "emailId": "18f3f3abc123"
}
```

### Create a label

Nesting is expressed with `/` — `Invoices/2026` appears as `2026` under `Invoices`.

```json
{
  "operation": "createLabel",
  "labelName": "Invoices/2026"
}
```

### List label IDs

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET",
  "refreshToken": "YOUR_REFRESH_TOKEN",
  "operation": "listLabels"
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

When full message scope is unavailable, the node falls back to metadata and sets `gmail.reducedFormat=true` on the parsed message. Attachment content is not available in this mode.

#### Attachment not sent, or arrives corrupt

Attachments must be supplied as `contentBase64`. The former `path` option was
removed: engine workers are stateless and horizontally scaled, so there is no
shared filesystem for a path to point at, and which worker runs the graph is not
predictable.

If the node errors with *is not valid base64*, the value bound to `contentBase64`
isn't base64 — a common cause is wiring a file path or an already-decoded string
into the field.

#### Attachment content is a `data:` URI

Supported — the `data:<mime>;base64,` prefix is stripped automatically. Whitespace
and line-wrapped base64 are also accepted.

<!-- /SECTION: troubleshooting -->

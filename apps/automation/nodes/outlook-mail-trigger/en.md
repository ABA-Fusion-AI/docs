---
node_id: "outlook-mail-trigger"
title: "Outlook Mail Trigger"
description: "Poll an Outlook mailbox through Microsoft Graph and start the workflow for each new email."
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-11-23"
author: "Fusion Team"
tags:
  - trigger
  - outlook
  - mail
  - email
  - polling
  - microsoft-graph
related_nodes:
  - outlook-mail
  - outlook-calendar-trigger
---

<!-- SECTION: overview -->
# Outlook Mail Trigger

> **Category:** Triggers &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

Start a workflow when a **new email** arrives in an **Outlook / Microsoft 365** mailbox. The node **polls** Microsoft Graph (`GET /messages`) at a fixed interval, remembers the messages it has already seen, and emits **one event per new message** in a simple format (subject, sender, recipients, dates, body).

### Use Cases

- **Auto-reply / triage:** Classify incoming emails with an LLM and reply or forward them with the Outlook Mail node.
- **Ticketing:** Create a ticket for each email received on a support address.
- **Alerts:** Notify a chat channel when an email matching a filter arrives.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

Use **one** of the two methods:

| Method | Fields | Mailbox |
|--------|--------|---------|
| Access token (delegated) | `accessToken` | The signed-in user (`/me`). Leave `userId` empty. |
| Client credentials (app-only) | `tenantId`, `clientId`, `clientSecret` + `userId` | The mailbox given in `userId`. |

The token needs `Mail.Read`. A Graph Explorer token expires after about 1 hour, so use **client credentials** for a trigger that runs for a long time.

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `accessToken` | `string` | One auth method | — | Microsoft Graph bearer token. |
| `tenantId` / `clientId` / `clientSecret` | `string` | One auth method | — | Azure app credentials (client credentials flow). |
| `baseUrl` | `string` | No | `https://graph.microsoft.com/v1.0` | Graph base URL. |
| `userId` | `string` | With client credentials | — | Mailbox email or user ID. |
| `intervalMs` | `number` | No | `15000` | Poll interval in milliseconds (minimum `1000`). |
| `emitOnStart` | `boolean` | No | `false` | `false`: the first poll only records existing messages, so only emails received **after** the start are emitted. `true`: existing messages are emitted at start too. |
| `messageFilters` | `object` | No | — | `top` (1-999), `skip`, `filter` (OData `$filter`, e.g. `from/emailAddress/address eq 'boss@example.com'`), `orderBy`, `select`. |
| `includeBody` | `boolean` | No | `false` | Also fetch the message body. |

> The trigger only looks at the messages returned by each poll (Graph returns **10** by default). If you receive many emails per interval, raise `messageFilters.top` and set `orderBy` to `receivedDateTime desc`.

> The list of seen messages is kept in memory: after a workflow restart, the first poll starts again from `emitOnStart`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This node has no inputs. It starts on its own when the workflow is started.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | One event per new message (see below). |
| `error` | `Error` | Configuration errors (missing authentication or `userId`). |

| Field | Description |
|-------|-------------|
| `type` | Always `"message"`. |
| `id` | Message ID (use it with the Outlook Mail node: `getMessage`, `replyToMessage`, ...). |
| `subject` | Subject. |
| `from` / `fromEmail` | Sender name and address. |
| `to`, `cc`, `bcc` | Arrays of `{ name, address }`. |
| `receivedDateTime`, `sentDateTime` | ISO dates. |
| `body`, `bodyType` | Body and `html` / `text`. |
| `bodyPreview` | Short text preview. |
| `raw` | The full Graph message. |

### Output Example

```json
{
  "type": "message",
  "id": "AQMkADAwATM0MDAAMS0wMjVl...",
  "subject": "New order #1042",
  "from": "Jane Doe",
  "fromEmail": "jane@example.com",
  "to": [{ "name": "Sales", "address": "sales@example.com" }],
  "cc": [],
  "bcc": [],
  "receivedDateTime": "2026-11-23T15:29:42Z",
  "sentDateTime": "2026-11-23T15:29:41Z",
  "body": "<html><body>Hello, please find the order ...</body></html>",
  "bodyType": "html",
  "bodyPreview": "Hello, please find the order ...",
  "raw": { "...": "..." }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Listen for new Outlook emails
```

### How it flows

1. **Outlook Mail Trigger:** Polls the mailbox and emits each new email.
2. **Log Node:** Displays the email.

### Example Configurations

#### Every new email, every 30 seconds

```json
{
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "intervalMs": 30000,
  "emitOnStart": false,
  "includeBody": true
}
```

#### Only unread emails from one sender (app-only)

```json
{
  "tenantId": "00000000-0000-0000-0000-000000000000",
  "clientId": "11111111-1111-1111-1111-111111111111",
  "clientSecret": "{{ secrets.AZURE_CLIENT_SECRET }}",
  "userId": "support@contoso.com",
  "messageFilters": {
    "top": 25,
    "filter": "isRead eq false and from/emailAddress/address eq 'client@example.com'"
  }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token and the client secret in Fusion's **Secrets** vault. Do not commit them into version control.

- Email content is untrusted input: validate it before using it in prompts, queries or commands.
- Keep `intervalMs` reasonable (15 s or more) to stay within Microsoft Graph throttling limits.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### The trigger never fires
- **Cause:** `emitOnStart` is `false`, so existing emails are ignored, or the token expired (poll errors are only logged).
- **Solution:** Send a new email after starting the workflow, and check that the token is still valid.

#### Old emails are emitted at start
- **Cause:** `emitOnStart` is `true`.
- **Solution:** Set it to `false`.

#### Some emails are missed
- **Cause:** More new emails than returned by one poll (10 by default).
- **Solution:** Increase `messageFilters.top` or shorten `intervalMs`.

#### `Either accessToken or (tenantId, clientId, clientSecret) must be provided`
- **Cause:** No authentication configured.
- **Solution:** Fill in `accessToken`, or the three client credentials fields (with `userId`).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-23 | Full documentation: polling behaviour, filters, output format, troubleshooting |

<!-- /SECTION: changelog -->

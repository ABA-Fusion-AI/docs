---
node_id: "outlook-mail-trigger"
title: "Outlook Mail Trigger"
description: "Poll Outlook messages and emit newly detected emails"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - outlook
  - mail
  - polling
related_nodes:
  - outlook-mail
---

<!-- SECTION: header -->

# Outlook Mail Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls Microsoft Graph messages and emits unseen messages.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Authentication

Provide either:

- `accessToken`, or
- `tenantId` + `clientId` + `clientSecret`

Optional:

- `baseUrl` (default: `https://graph.microsoft.com/v1.0`)

### Parameters

| Parameter        | Type      | Required | Default | Description                                  |
| ---------------- | --------- | -------- | ------- | -------------------------------------------- |
| `intervalMs`     | `number`  | ❌ No    | `15000` | Poll interval (min `1000`)                   |
| `emitOnStart`    | `boolean` | ❌ No    | `false` | If `false`, first poll warms cache only      |
| `messageFilters` | `object`  | ❌ No    | —       | `top`, `skip`, `filter`, `orderBy`, `select` |
| `includeBody`    | `boolean` | ❌ No    | `false` | Adds body to selected fields                 |

### Output

Each emitted item is normalized to:

- `type`, `id`, `subject`
- `from`, `fromEmail`
- `to`, `cc`, `bcc` arrays
- `receivedDateTime`, `sentDateTime`
- `body`, `bodyType`, `bodyPreview`
- `raw` (original message payload)

<!-- /SECTION: configuration -->

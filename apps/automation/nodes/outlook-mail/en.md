---
node_id: "outlook-mail"
title: "Outlook Mail"
description: "Microsoft Outlook mail operations via Microsoft Graph"
category: "integrations"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - outlook
  - mail
  - microsoft-graph
related_nodes:
  - outlook-mail-trigger
  - outlook-calendar
---

<!-- SECTION: header -->

# Outlook Mail

> **Category:** Integrations | **Type:** Action Node

Mail operations for Microsoft 365: message read/send/update/delete/reply/forward and folder retrieval.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Authentication

Provide either:

- `accessToken`, or
- `tenantId` + `clientId` + `clientSecret` (client credentials flow)

Optional:

- `baseUrl` (default: `https://graph.microsoft.com/v1.0`)

### Operations

`operation` supports:

- `getMessages`
- `getMessage`
- `sendMail`
- `createMessage`
- `updateMessage`
- `deleteMessage`
- `replyToMessage`
- `replyAllToMessage`
- `forwardMessage`
- `getMailFolders`
- `getMailFolder`

### Key Parameters

| Parameter           | Used By                                                                                                 | Notes                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `messageId`         | `getMessage`, `updateMessage`, `deleteMessage`, `replyToMessage`, `replyAllToMessage`, `forwardMessage` | Required                                              |
| `messageFilters`    | `getMessages`                                                                                           | `top`, `skip`, `filter`, `orderBy`, `select`          |
| `includeBody`       | `getMessages`                                                                                           | Adds body fields to `$select`                         |
| `emailData`         | `sendMail`, `createMessage`                                                                             | Requires `to` and `subject`                           |
| `messageUpdateData` | `updateMessage`                                                                                         | `subject`, `body`, `bodyType`, `isRead`, `flagStatus` |
| `replyData`         | `replyToMessage`, `replyAllToMessage`, `forwardMessage`                                                 | `forwardMessage` requires `replyData.to`              |
| `folderId`          | `getMailFolder`                                                                                         | Required                                              |

### Outputs

- `getMessages`, `getMessage`, `createMessage`, `updateMessage`, folder operations: Microsoft Graph response payload
- `sendMail`: success response (often empty payload from Graph)
- `deleteMessage`: `{ "success": true }`
- Reply/forward operations: Graph response payload

<!-- /SECTION: configuration -->

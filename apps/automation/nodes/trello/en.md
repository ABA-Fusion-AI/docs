---
node_id: "trello"
title: "Trello"
description: "Manage Trello boards, lists, cards, comments, checklists, labels, attachments and board members through the Trello REST API."
category: "integrations"
subcategory: "project-management"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - trello
  - kanban
  - boards
  - cards
  - project-management
related_nodes:
  - wekan
  - mondaycom
  - jira
  - clickup
---

<!-- SECTION: overview -->
# Trello

> **Category:** Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Automate **Trello** from a Fusion workflow with the Trello REST API (`https://api.trello.com/1`). The node offers **34 operations** in 8 resources:

| Resource | Operations |
|----------|------------|
| Boards | `getBoard`, `createBoard`, `updateBoard`, `deleteBoard` |
| Board members | `getBoardMembers`, `addBoardMember`, `updateBoardMember`, `removeBoardMember` |
| Lists | `getList`, `getBoardLists`, `createList`, `updateList`, `deleteList` |
| Cards | `getCard`, `createCard`, `updateCard`, `deleteCard` |
| Comments | `getCardComments`, `addCardComment`, `updateCardComment`, `deleteCardComment` |
| Checklists | `getChecklist`, `createChecklist`, `updateChecklist`, `deleteChecklist` |
| Labels | `getLabel`, `getBoardLabels`, `createLabel`, `updateLabel`, `deleteLabel` |
| Attachments | `getAttachments`, `createAttachment`, `getAttachment`, `deleteAttachment` |

### Use Cases

- **Task intake:** Create a card for each form submission, support ticket or email.
- **Status sync:** Move or update cards when the status changes in another tool (GitHub, Jira, CRM).
- **Project setup:** Create a board with its lists, labels and checklists for each new project.
- **Collaboration:** Add comments, attachments or members automatically.
- **Reporting:** Read the lists and cards of a board to build a summary.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | Yes | Trello API key. |
| `apiToken` | `string` | Yes | Trello API token. |
| `operation` | `enum` | Yes | Operation to perform. |
| `debug` | `boolean` | No | Logs extra information in the console. Default `false`. |

**Getting the key and token**

1. Go to `https://trello.com/power-ups/admin` and create a Power-Up (a name and a workspace are enough).
2. Open the **API Key** tab and generate a key.
3. Click the **Token** link next to the key and allow access to get a token.
4. Store both in Fusion Secrets.

### Operations & Parameters

#### Boards

| Operation | Required | Optional |
|-----------|----------|----------|
| `getBoard` | `boardId` | `fields`, `actions`, `cards`, `members` |
| `createBoard` | `boardName` | `boardDesc`, `idOrganization` (workspace), `prefs_permissionLevel` (`org` / `private` / `public`), `defaultLists` |
| `updateBoard` | `boardId` | `boardName`, `boardDesc`, `boardClosed` (archive), `boardSubscribed` |
| `deleteBoard` | `boardId` | — |

#### Board members

| Operation | Required | Optional |
|-----------|----------|----------|
| `getBoardMembers` | `boardId` | `fields`, `filter` |
| `addBoardMember` | `boardId`, `memberEmail`, `memberFullName` | `memberType` (`admin` / `normal` / `observer`) |
| `updateBoardMember` | `boardId`, `memberId`, `memberType` | — |
| `removeBoardMember` | `boardId`, `memberId` | — |

#### Lists

| Operation | Required | Optional |
|-----------|----------|----------|
| `getList` | `listId` | `fields`, `cards` |
| `getBoardLists` | `boardId` | `fields`, `filter`, `cards` |
| `createList` | `boardId`, `listName` | `listPos` (`top`, `bottom` or a number), `idListSource` (list to copy) |
| `updateList` | `listId` | `listName`, `listClosed`, `listPos`, `listSubscribed` |
| `deleteList` | `listId` | — |

> Trello has no endpoint to delete a list: `deleteList` **archives** it (`closed: true`).

#### Cards

| Operation | Required | Optional |
|-----------|----------|----------|
| `getCard` | `cardId` | `fields`, `actions`, `attachments`, `members`, `checklists` |
| `createCard` | `listId`, `cardName` | `cardDesc`, `cardPos`, `cardDue` (ISO date), `idMembers`, `idLabels` |
| `updateCard` | `cardId` | `cardName`, `cardDesc`, `cardClosed`, `listId` (moves the card), `cardPos`, `cardDue`, `idMembers`, `idLabels` |
| `deleteCard` | `cardId` | — |

#### Comments

| Operation | Required | Optional |
|-----------|----------|----------|
| `getCardComments` | `cardId` | `commentFilter` (e.g. `commentCard`), `commentPage` |
| `addCardComment` | `cardId`, `commentText` | — |
| `updateCardComment` | `cardId`, `commentId`, `commentText` | — |
| `deleteCardComment` | `commentId` | — |

#### Checklists

| Operation | Required | Optional |
|-----------|----------|----------|
| `getChecklist` | `checklistId` | `fields` |
| `createChecklist` | `cardId` | `checklistName`, `checklistPos`, `idChecklistSource` (checklist to copy) |
| `updateChecklist` | `checklistId` | `checklistName`, `checklistPos` |
| `deleteChecklist` | `checklistId` | — |

#### Labels

| Operation | Required | Optional |
|-----------|----------|----------|
| `getLabel` | `labelId` | `fields` |
| `getBoardLabels` | `boardId` | `fields`, `labelLimit` |
| `createLabel` | `boardId`, `labelName`, `labelColor` | — |
| `updateLabel` | `labelId` | `labelName`, `labelColor` |
| `deleteLabel` | `labelId` | — |

Colors: `yellow`, `purple`, `blue`, `red`, `green`, `orange`, `black`, `sky`, `pink`, `lime`.

#### Attachments

| Operation | Required | Optional |
|-----------|----------|----------|
| `getAttachments` | `cardId` | `fields`, `filter` |
| `createAttachment` | `cardId`, `attachmentUrl` | `attachmentName`, `setCover` |
| `getAttachment` | `cardId`, `attachmentId` | `fields` |
| `deleteAttachment` | `cardId`, `attachmentId` | — |

### Where to find the IDs

- **Board / card:** Add `.json` to the board or card URL in the browser (e.g. `https://trello.com/c/AbC12xYz.json`); the first `id` field is the ID. The short code in the URL (`AbC12xYz`) can also be used as `cardId` / `boardId`.
- **Lists, labels, members:** Use `getBoardLists`, `getBoardLabels`, `getBoardMembers`.
- **Comments, checklists, attachments:** Returned by the matching `create…` / `get…` operations (`id` field).

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
| `success` | `object` / `array` | JSON response of the Trello API. |
| `error` | `Error` | Emitted on missing fields or API errors. |

### Output Examples

#### `createCard`

```json
{
  "id": "6afc74b5f6560ca8804ef014",
  "name": "Carte Fusion",
  "desc": "Carte de test",
  "idBoard": "6afc73fd57ef5243610b71b8",
  "idList": "6afc7481a4f722f043d7c784",
  "closed": false,
  "url": "https://trello.com/c/AbC12xYz/1-carte-fusion",
  "badges": { "comments": 0, "attachments": 0 }
}
```

#### `getBoardMembers`

```json
[
  { "id": "6636c8425588509d1f0d8124", "fullName": "Karam Yassine", "username": "karamyassine" }
]
```

#### `deleteCard` / `deleteLabel` / `deleteAttachment` / `deleteChecklist`

```json
{ "limits": {} }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Create a Trello card
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Trello Node:** Runs the selected operation.
3. **Log Node:** Displays the Trello response (connected to both `success` and `error`).

A typical chain: **createBoard → getBoardLists → createCard → addCardComment / createChecklist / createAttachment**, passing the IDs with expressions such as `{{ input.id }}`.

### Example Configuration

```json
{
  "apiKey": "{{ secrets.TRELLO_API_KEY }}",
  "apiToken": "{{ secrets.TRELLO_API_TOKEN }}",
  "operation": "createCard",
  "listId": "6afc7481a4f722f043d7c784",
  "cardName": "New lead: {{ input.company }}",
  "cardDesc": "{{ input.message }}",
  "cardDue": "2026-12-01T09:00:00.000Z"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Trello `apiKey` and `apiToken` in Fusion's **Secrets** vault. Do not commit them into version control.

- The token gives the same rights as your Trello account: generate it with a dedicated account for automations if possible.
- The key and token are sent as URL query parameters: avoid logging full request URLs.
- `deleteBoard`, `deleteCard` and the other delete operations are permanent; `addBoardMember` sends a real invitation email.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `<field> is required`
- **Cause:** A required parameter of the selected operation is empty (e.g. `labelColor is required` for `createLabel`).
- **Solution:** Fill in the field; see the tables above.

#### `Unauthorized - Check your API key and token. Trello API error (401): invalid key`
- **Cause:** Wrong or incomplete API key (it has 32 characters).
- **Solution:** Copy the key again from the Power-Up admin page.

#### `... (401): invalid token` / `unauthorized permission requested`
- **Cause:** Expired or revoked token, or token without write access.
- **Solution:** Generate a new token from the **Token** link of your API key.

#### `Not Found - Resource not found. Trello API error (404)`
- **Cause:** Wrong ID, or the object was deleted.
- **Solution:** Get the IDs again with the `get…` operations.

#### `Bad Request - Check your data. Trello API error (400)`
- **Cause:** Invalid value (e.g. wrong date format in `cardDue`, unknown color).
- **Solution:** Use ISO dates and the listed colors.

#### `Rate Limit - Too many requests (429)`
- **Cause:** Trello limits API calls per key and token.
- **Solution:** Slow down the workflow and retry.

#### A deleted list is still visible as archived
- **Cause:** Expected: `deleteList` archives the list, Trello cannot delete lists.
- **Solution:** Use `updateList` with `listClosed: false` to restore it.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: 34 operations, authentication, parameters, output examples, troubleshooting |

<!-- /SECTION: changelog -->

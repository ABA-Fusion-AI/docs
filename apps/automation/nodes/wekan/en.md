---
node_id: "wekan"
title: "Wekan"
description: "Manage boards, lists, swimlanes, cards and comments in a Wekan kanban server through its REST API."
category: "integrations"
subcategory: "project-management"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - wekan
  - kanban
  - boards
  - cards
  - project-management
  - self-hosted
related_nodes:
  - trello
  - mondaycom
  - jira
  - clickup
---

<!-- SECTION: overview -->
# Wekan

> **Category:** Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage a **Wekan** server (open-source, self-hosted kanban) from a Fusion workflow through its REST API. The node logs in with a Wekan username and password, then runs one of 12 operations:

- **Boards:** `listBoards`, `getBoard`, `createBoard`
- **Lists & swimlanes:** `listLists`, `createList`, `listSwimlanes`
- **Cards:** `listCards`, `getCard`, `createCard`, `updateCard`, `deleteCard`
- **Comments:** `addComment`

### Use Cases

- **Ticket intake:** Create a card each time a form is submitted, an email arrives or an alert fires.
- **Status sync:** Update a card's title or description when a task changes in another tool.
- **Reporting:** List the cards of a list to build a daily summary or feed a dashboard.
- **Board provisioning:** Create a board and its lists for each new project.
- **Audit trail:** Add a comment to a card each time a workflow processes it.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | Yes | — | Wekan base URL, e.g. `https://wekan.example.com` (a trailing `/` is accepted). |
| `username` | `string` | Yes | — | Wekan username. |
| `password` | `string` | Yes | — | Wekan password. Supports expressions. |
| `operation` | `enum` | Yes | `listBoards` | Operation to perform (see table below). |
| `boardId` | `string` | Depends on operation | — | Board ID. |
| `listId` | `string` | Depends on operation | — | List ID. |
| `cardId` | `string` | Depends on operation | — | Card ID. |
| `swimlaneId` | `string` | For `createCard` | — | Destination swimlane. Required by Wekan to create a card; use `listSwimlanes` to find it. |
| `title` | `string` | Depends on operation | — | Title of the board, list or card. |
| `description` | `string` | No | — | Card description (`createCard`, `updateCard`). |
| `permission` | `enum` | No | `private` | Board visibility for `createBoard`: `private` or `public`. |
| `comment` | `string` | For `addComment` | — | Text of the comment. |
| `authorId` | `string` | No | Authenticated user | Author of the card or comment (`createCard`, `addComment`). |

### Operations

| Operation | Wekan Endpoint | Required Fields |
|-----------|----------------|-----------------|
| `listBoards` | `GET /api/users/{userId}/boards` | — |
| `getBoard` | `GET /api/boards/{boardId}` | `boardId` |
| `createBoard` | `POST /api/boards` | `title` |
| `listLists` | `GET /api/boards/{boardId}/lists` | `boardId` |
| `createList` | `POST /api/boards/{boardId}/lists` | `boardId`, `title` |
| `listSwimlanes` | `GET /api/boards/{boardId}/swimlanes` | `boardId` |
| `listCards` | `GET /api/boards/{boardId}/lists/{listId}/cards` | `boardId`, `listId` |
| `getCard` | `GET /api/boards/{boardId}/lists/{listId}/cards/{cardId}` | `boardId`, `listId`, `cardId` |
| `createCard` | `POST /api/boards/{boardId}/lists/{listId}/cards` | `boardId`, `listId`, `swimlaneId`, `title` |
| `updateCard` | `PUT /api/boards/{boardId}/lists/{listId}/cards/{cardId}` | `boardId`, `listId`, `cardId` + `title` and/or `description` |
| `deleteCard` | `DELETE /api/boards/{boardId}/lists/{listId}/cards/{cardId}` | `boardId`, `listId`, `cardId` |
| `addComment` | `POST /api/boards/{boardId}/cards/{cardId}/comments` | `boardId`, `cardId`, `comment` |

### Authentication

The node logs in once at start-up with `POST /users/login` (username + password) and uses the returned token for every call (`Authorization: Bearer` + `X-User-Id` headers).

1. Use an existing Wekan account, or create one on your Wekan server.
2. Make sure the server API is enabled (`WITH_API=true` in the Wekan configuration).
3. Fill in `host`, `username` and `password` (store the password in Fusion Secrets).

### Where to find the IDs

Chain the operations to get the IDs:

1. `listBoards` → `_id` of the board
2. `listLists` and `listSwimlanes` → `_id` of the list and of the swimlane
3. `listCards` → `_id` of the card

The board ID also appears in the board URL: `https://<host>/b/<boardId>/<board-slug>`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Upstream values can be used through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `array` | JSON response of the Wekan API. Empty responses (e.g. `deleteCard`) return `{ "success": true }`. |
| `error` | `Error` | Emitted on missing fields, authentication failure or API errors. |

### Output Examples

#### `listBoards`

```json
[
  { "_id": "9WvKzX3bqPLEc7Ruy", "title": "Fusion Tests" },
  { "_id": "fXhGJk2LmPq8sYtWd", "title": "Templates" }
]
```

#### `listLists` / `listSwimlanes`

```json
[
  { "_id": "aB3dE5fG7hJ9kL1mN", "title": "To Do" },
  { "_id": "pQ2rS4tU6vW8xY0zA", "title": "Done" }
]
```

#### `createCard`

```json
{
  "_id": "Zx9Yw8Vu7Ts6Rq5Po"
}
```

#### `deleteCard`

```json
{
  "_id": "Zx9Yw8Vu7Ts6Rq5Po"
}
```

> Depending on the Wekan version, `deleteCard` may answer with an empty body; the node then returns `{ "success": true }`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Wekan boards
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Wekan Node:** Logs in and runs the selected operation (`listBoards`).
3. **Log Node:** Displays the boards returned by Wekan.

### Example Configuration

```json
{
  "host": "https://wekan.example.com",
  "username": "fusion",
  "password": "{{ secrets.WEKAN_PASSWORD }}",
  "operation": "createCard",
  "boardId": "9WvKzX3bqPLEc7Ruy",
  "listId": "aB3dE5fG7hJ9kL1mN",
  "swimlaneId": "sW1mL4n3Id0000000",
  "title": "New request from {{ input.name }}",
  "description": "{{ input.message }}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Wekan `password` in Fusion's **Secrets** vault. Do not commit hardcoded passwords into version control.

- Use a dedicated Wekan account for automations, with access only to the boards it needs.
- Always use `https://` for `host`: the password is sent at login.
- `deleteCard` is destructive: restrict who can run workflows that use it.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `<field> is required for <operation>`
- **Cause:** A field needed by the selected operation is empty (e.g. `swimlaneId is required for createCard`).
- **Solution:** Fill in the field; see the *Operations* table.

#### `updateCard requires at least one of title or description`
- **Cause:** `updateCard` was run without `title` nor `description`.
- **Solution:** Provide at least one of them.

#### `Wekan authentication failed (401): ...`
- **Cause:** Wrong username or password.
- **Solution:** Check the credentials by logging in on the Wekan web interface.

#### `Wekan authentication failed (404): <html>...`
- **Cause:** `host` does not point to a Wekan server (wrong URL, or a path was added).
- **Solution:** Use the base URL only, e.g. `https://wekan.example.com`.

#### `Wekan authentication failed: no token in response`
- **Cause:** The server answered without a token (API disabled or unexpected response).
- **Solution:** Enable the API on the server (`WITH_API=true`).

#### `Wekan API error 401: ...` / `403`
- **Cause:** The user has no access to this board, or the session expired.
- **Solution:** Add the user to the board, then run the workflow again (the node logs in again on start).

#### `Wekan API error 404` / `500` on cards
- **Cause:** A wrong `boardId`, `listId`, `cardId` or `swimlaneId`, or IDs from different boards.
- **Solution:** Get the IDs again with `listBoards`, `listLists`, `listSwimlanes` and `listCards`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: operations, authentication, parameter reference, output examples, and workflow integration |

<!-- /SECTION: changelog -->

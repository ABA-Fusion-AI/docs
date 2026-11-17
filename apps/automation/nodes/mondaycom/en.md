---
node_id: "mondaycom"
title: "Monday.com"
description: "Interact with the Monday.com GraphQL API to manage boards, items, workspaces, and users."
category: "integrations"
subcategory: "project-management"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - monday
  - monday.com
  - project-management
  - boards
  - items
  - graphql
related_nodes:
  - jira
  - trello
  - asana
  - clickup
---

<!-- SECTION: overview -->
# Monday.com

> **Category:** Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage your **Monday.com** account from a Fusion workflow through the Monday.com GraphQL API (`https://api.monday.com/v2`). The node covers 14 operations grouped in four resources:
- **Boards:** `getBoards`, `getBoard`, `createBoard`, `updateBoard`, `deleteBoard`
- **Items:** `getItems`, `getItem`, `createItem`, `updateItem`, `deleteItem`
- **Workspaces:** `getWorkspaces`, `getWorkspace`
- **Users:** `getCurrentUser`, `getUsers`

### Use Cases

- **Lead & ticket intake:** Create an item on a board each time a form is submitted, an email arrives or a ticket is opened.
- **Status sync:** Update an item's columns when a task changes state in another tool (Jira, GitHub, CRM).
- **Reporting:** Read boards and items to build a weekly summary or feed a dashboard.
- **Board provisioning:** Create a new board from a template for each new project or client.
- **Team directory:** List users and workspaces for audits or onboarding automations.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiToken` | `string` | Yes | — | Monday.com API token. Supports expressions. |
| `apiVersion` | `string` | No | `2026-07` | Monday.com API version sent in the `API-Version` header. |
| `operation` | `enum` | Yes | `getBoards` | Operation to perform (see table below). |
| `boardId` | `number` | For `getBoard`, `updateBoard`, `deleteBoard` | — | Board ID. |
| `boardFilters` | `object` | No | — | Filters for `getBoards` (see below). |
| `boardData` | `object` | For `createBoard`, `updateBoard` | — | Board fields (see below). |
| `itemId` | `number` | For `getItem`, `updateItem`, `deleteItem` | — | Item ID. |
| `itemFilters` | `object` | No | — | Filters for `getItems` (see below). |
| `itemData` | `object` | For `createItem`, `updateItem` | — | Item fields (see below). |
| `workspaceId` | `number` | For `getWorkspace` | — | Workspace ID. |
| `userFilters` | `object` | No | — | Filters for `getUsers` (see below). |

### Operations

| Operation | Description | Required Fields |
|-----------|-------------|-----------------|
| `getBoards` | List boards with their groups and up to 100 items each | — |
| `getBoard` | Get one board with its groups, columns and items | `boardId` |
| `createBoard` | Create a board | `boardData.boardName` |
| `updateBoard` | Rename, change description, archive or delete a board | `boardId` + at least one `boardData` field |
| `deleteBoard` | Delete a board | `boardId` |
| `getItems` | List items, from given boards or across the account | — |
| `getItem` | Get one item with its column values and updates | `itemId` |
| `createItem` | Create an item on a board | `itemData.boardId`, `itemData.itemName` |
| `updateItem` | Rename, change column values, or move an item | `itemId` + at least one `itemData` field |
| `deleteItem` | Delete an item | `itemId` |
| `getWorkspaces` | List workspaces | — |
| `getWorkspace` | Get one workspace with its owners and subscribers | `workspaceId` |
| `getCurrentUser` | Get the user who owns the API token | — |
| `getUsers` | List users | — |

### Object Fields

**`boardFilters`** (`getBoards`)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | `number` (1–100) | `25` | Number of boards per page. |
| `page` | `number` | `1` | Page number. |
| `ids` | `number[]` | — | Only return these boards. |
| `boardKind` | `public` \| `private` \| `share` | — | Filter by board type. |
| `state` | `all` \| `active` \| `archived` \| `deleted` | `active` | Filter by board state. |

**`boardData`** (`createBoard`, `updateBoard`)

| Field | Type | Used by | Description |
|-------|------|---------|-------------|
| `boardName` | `string` | create, update | Board name. Required for `createBoard`. |
| `boardKind` | `public` \| `private` \| `share` | create | Board type. Default `private`. Cannot be changed after creation. |
| `workspaceId` | `number` | create | Workspace to create the board in. |
| `templateId` | `number` | create | Template to create the board from. |
| `description` | `string` | create, update | Board description. |
| `state` | `archived` \| `deleted` | update | Archives or deletes the board. `active` is not supported. |

**`itemFilters`** (`getItems`)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `boardIds` | `number[]` | — | Return items of these boards. When set, `itemIds` and `page` are ignored. |
| `itemIds` | `number[]` | — | Only return these items. |
| `limit` | `number` (1–100) | `25` | Number of items. |
| `page` | `number` | `1` | Page number (only without `boardIds`). |

**`itemData`** (`createItem`, `updateItem`)

| Field | Type | Description |
|-------|------|-------------|
| `boardId` | `number` | Board of the new item (`createItem`). For `updateItem`, a different board moves the item to it. |
| `groupId` | `string` | Group of the item (e.g. `topics`). For `updateItem`, moves the item to this group. |
| `itemName` | `string` | Item name. Required for `createItem`. |
| `columnValues` | `object` | Column values keyed by column ID, e.g. `{ "status": { "label": "Done" }, "date4": { "date": "2026-11-20" } }`. |

**`userFilters`** (`getUsers`)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ids` | `number[]` | — | Only return these users. |
| `kind` | `all` \| `non_guests` \| `guests` \| `non_pending` | `all` | Filter by user type. `non_pending` returns active users only. |
| `limit` | `number` (1–100) | `25` | Number of users per page. |
| `page` | `number` | `1` | Page number. |

### Authentication & API Token

1. Log in to **Monday.com**.
2. Click your avatar → **Developers** (or **Administration → Connections → API**).
3. Open **My Access Tokens** and click **Show** / **Copy**.
4. Paste the token into the `apiToken` field (or store it in Fusion Secrets).

### Where to find the IDs

- **Board ID:** In the board URL `https://<account>.monday.com/boards/<boardId>`.
- **Item ID:** Open the item; the URL ends with `/pulses/<itemId>`.
- **Group ID / Column ID:** Returned by `getBoard` (`groups[].id`, `columns[].id`).
- **Workspace ID:** Returned by `getWorkspaces`.

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
| `success` | `object` | GraphQL response of Monday.com (`{ "data": { ... } }`). `deleteBoard` and `deleteItem` return `{ "success": true }`. |
| `error` | `Error` | Emitted on validation errors, HTTP errors or GraphQL errors. |

### Output Examples

#### `getBoards`

```json
{
  "data": {
    "boards": [
      {
        "id": "1234567890",
        "name": "Marketing Tasks",
        "description": null,
        "state": "active",
        "board_kind": "public",
        "workspace": { "id": "987654", "name": "Main workspace" },
        "groups": [{ "id": "topics", "title": "To Do" }],
        "items_page": {
          "items": [{ "id": "1122334455", "name": "Write blog post" }]
        }
      }
    ]
  },
  "account_id": 12345678
}
```

#### `createItem`

```json
{
  "data": {
    "create_item": {
      "id": "1122334466",
      "name": "New lead: ACME Corp",
      "state": "active",
      "board": { "id": "1234567890" }
    }
  },
  "account_id": 12345678
}
```

#### `deleteItem` / `deleteBoard`

```json
{
  "success": true
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Monday.com boards
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Monday.com Node:** Runs the selected operation (`getBoards`).
3. **Log Node:** Displays the boards returned by Monday.com.

### Example Configuration

```json
{
  "apiToken": "{{ secrets.MONDAY_API_TOKEN }}",
  "operation": "createItem",
  "itemData": {
    "boardId": 1234567890,
    "groupId": "topics",
    "itemName": "New lead: {{ input.company }}",
    "columnValues": {
      "status": { "label": "Working on it" },
      "email": { "email": "{{ input.email }}", "text": "{{ input.email }}" }
    }
  }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your Monday.com `apiToken` in Fusion's **Secrets** vault. Do not commit hardcoded tokens into version control.

- The personal API token has the same permissions as your Monday.com user: use a dedicated user with limited access for automations.
- `deleteBoard`, `deleteItem` and `updateBoard` with `state: deleted` are destructive: restrict who can run these workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Monday.com operation failed: Board ID is required` / `Item ID is required` / `Workspace ID is required`
- **Cause:** The selected operation needs an ID that was left empty.
- **Solution:** Fill in `boardId`, `itemId` or `workspaceId`.

#### `Monday.com operation failed: Board name is required`
- **Cause:** `createBoard` was run without `boardData.boardName`.
- **Solution:** Set a board name.

#### `Monday.com operation failed: Board ID and item name are required`
- **Cause:** `createItem` was run without `itemData.boardId` or `itemData.itemName`.
- **Solution:** Fill in both fields.

#### `Monday.com does not allow changing board kind after creation`
- **Cause:** `boardData.boardKind` was set on `updateBoard`.
- **Solution:** Remove `boardKind`; create a new board if a different type is needed.

#### `Monday.com API does not support restoring a board to active state through update_board`
- **Cause:** `boardData.state` was set to `active` on `updateBoard`.
- **Solution:** Restore the board from the Monday.com interface.

#### `No fields to update`
- **Cause:** `updateBoard` or `updateItem` was run without any field to change.
- **Solution:** Provide at least one field in `boardData` / `itemData`.

#### `Monday.com API error (401)` / `Not Authenticated`
- **Cause:** The API token is invalid or was regenerated.
- **Solution:** Copy a new token from **Developers → My Access Tokens**.

#### `Monday.com API error: ... invalid value ...` on `columnValues`
- **Cause:** A column ID does not exist on the board or its value has the wrong format.
- **Solution:** Get the column IDs and types with `getBoard`, and use the format expected by each column type.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: operations, parameter reference, output schemas, and workflow integration |

<!-- /SECTION: changelog -->

---
node_id: "ms-todo"
title: "Microsoft To Do"
description: "Manage Microsoft To Do lists and tasks"
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-09"
author: "Fusion Team"
tags:
  - ms-todo
  - microsoft-graph
  - tasks
  - productivity
  - integration
  - peer-only
related_nodes:
  - microsoft-todo
  - http-request
  - function
---

<!-- SECTION: overview -->
# Microsoft To Do

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage lists and tasks in [Microsoft To Do](https://to-do.office.com) through the Microsoft Graph API: list, create and delete task lists, and list, create, update, complete and delete tasks. The node calls `https://graph.microsoft.com/v1.0/me/todo` with a Microsoft OAuth 2.0 access token, so it works on the To Do account of the signed-in user (personal Microsoft account or Microsoft 365 work account).

### Use Cases

- **Task creation from events:** Create a To Do task when a form is submitted, an e-mail arrives, or a ticket is opened.
- **Task maintenance:** Update the title, notes, importance or due date of a task as a process evolves.
- **Workflow closure:** Mark a task as completed when the related process finishes.
- **List lifecycle:** Create a list for a project and delete it when the project is closed.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `getLists` | Operation to perform (see below). |
| `accessToken` | `string` | Yes | — | Microsoft Graph OAuth 2.0 access token with the `Tasks.ReadWrite` permission. Supports expressions. |
| `listId` | `string` | For all operations except `getLists` and `createList` | — | Task list ID. Supports expressions. |
| `taskId` | `string` | For `updateTask`, `deleteTask`, `completeTask` | — | Task ID. Supports expressions. |
| `title` | `string` | For `createList` and `createTask` | — | List name (`createList`) or task title (`createTask`, `updateTask`). Supports expressions. |
| `body` | `string` | No | — | Task notes, plain text. Used by `createTask` and `updateTask`. Supports expressions. |
| `dueDateTime` | `string` | No | — | Due date, ISO format without time zone (e.g. `2026-12-31T00:00:00`), interpreted as UTC. Used by `createTask` and `updateTask`. Supports expressions. |
| `importance` | `string` | No | — | `low`, `normal`, or `high`. Used by `createTask` and `updateTask`. Supports expressions. |

### Available Operations

| Operation | Description | Graph request |
|-----------|-------------|---------------|
| `getLists` | List all task lists. | `GET /me/todo/lists` |
| `createList` | Create a task list named `title`. | `POST /me/todo/lists` |
| `deleteList` | Delete a task list and all its tasks. | `DELETE /me/todo/lists/{listId}` |
| `getTasks` | List the tasks of a list. | `GET /me/todo/lists/{listId}/tasks` |
| `createTask` | Create a task with `title`, `body`, `importance`, `dueDateTime`. | `POST /me/todo/lists/{listId}/tasks` |
| `updateTask` | Update the fields that are filled in (`title`, `body`, `importance`, `dueDateTime`). | `PATCH /me/todo/lists/{listId}/tasks/{taskId}` |
| `completeTask` | Set the task status to `completed`. | `PATCH /me/todo/lists/{listId}/tasks/{taskId}` |
| `deleteTask` | Delete a task. | `DELETE /me/todo/lists/{listId}/tasks/{taskId}` |

All parameters are shown for every operation. The node does not check `listId` and `taskId` before calling the API: if one is missing, Microsoft Graph returns an error (usually `400` or `404`). `getTasks` returns Microsoft Graph's default page size; there is no limit parameter.

> `deleteList` permanently deletes the list **and all its tasks**. Double-check `listId` before running it.

### Authentication

Every request sends `Authorization: Bearer <accessToken>`. The token must be a **delegated** Microsoft Graph token with the `Tasks.ReadWrite` permission (`Tasks.Read` is enough for read-only operations). The node does not refresh the token: when it expires (usually after about 1 hour), provide a new one.

### Getting an Access Token

**Quick test with Graph Explorer:**

1. Open [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer) and sign in with a Microsoft account (a free personal account such as Outlook.com works).
2. Open **Modify permissions**, find **Tasks.ReadWrite**, and click **Consent**.
3. Run `GET https://graph.microsoft.com/v1.0/me/todo/lists` to check access.
4. Copy the token from the **Access token** tab and paste it into `accessToken`.

**Production:** register an application in **Microsoft Entra ID → App registrations**, add the delegated permission `Tasks.ReadWrite`, and obtain tokens with the OAuth 2.0 authorization code flow (refresh the token before each run, e.g. with an HTTP Request node).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. The incoming data is not used directly; use expressions in the parameters to pass upstream values. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Microsoft Graph response (see examples). `deleteList` and `deleteTask` return `{ "success": true }`. |
| `error` | `Error` | `Microsoft To Do error: <response>`, e.g. `Microsoft To Do error: {"error":{"code":"InvalidAuthenticationToken",...}}`. |

### Output Examples

#### `getLists`

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('user%40example.com')/todo/lists",
  "value": [
    {
      "displayName": "Tasks",
      "isOwner": true,
      "isShared": false,
      "wellknownListName": "defaultList",
      "id": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgAuAAAD..."
    }
  ]
}
```

#### `updateTask`

```json
{
  "importance": "low",
  "status": "notStarted",
  "title": "Prepare the monthly report (updated)",
  "lastModifiedDateTime": "2026-10-09T10:23:12.1234567Z",
  "id": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgBGAAAD...",
  "body": {
    "content": "New notes",
    "contentType": "text"
  },
  "dueDateTime": {
    "dateTime": "2027-01-15T00:00:00.0000000",
    "timeZone": "UTC"
  }
}
```

#### `deleteTask` / `deleteList`

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
title: Use Microsoft To Do in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Microsoft To Do Node:** Runs the configured operation (e.g. `getLists`).
3. **Log Node:** Displays the Microsoft Graph response.

### Example Configurations

#### Create a task

```json
{
  "operation": "createTask",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "listId": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgAuAAAD...",
  "title": "Call back {{ input.customerName }}",
  "body": "Request received from {{ input.email }}",
  "importance": "high",
  "dueDateTime": "2026-12-31T00:00:00"
}
```

#### Postpone a task

```json
{
  "operation": "updateTask",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "listId": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgAuAAAD...",
  "taskId": "{{ input.taskId }}",
  "dueDateTime": "2027-01-15T00:00:00"
}
```

### Common Patterns

- **Webhook → Microsoft To Do (`createTask`):** Turn incoming requests into tasks.
- **Cron → Microsoft To Do (`getTasks`) → Function → Notification:** Send a daily summary of open tasks.
- **Microsoft To Do (`getLists`) → Function:** Find a list ID by its `displayName` before creating tasks in it.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `accessToken` in Fusion's **Secrets** system. Never paste it directly into workflow parameters or commit it to version control.

- Request only the permissions the workflow needs (`Tasks.Read` for read-only use).
- Access tokens are short-lived; tokens copied from Graph Explorer are meant for testing only.
- `deleteList` and `deleteTask` cannot be undone.
- Task titles and notes can contain personal data; avoid logging full responses in shared workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Microsoft To Do error: ... InvalidAuthenticationToken`
- **Cause:** The access token is missing, malformed, or expired (tokens usually last about 1 hour).
- **Solution:** Get a new token and update `accessToken`.

#### `Microsoft To Do error: ... Forbidden` / `AccessDenied`
- **Cause:** The token does not include the `Tasks.ReadWrite` (or `Tasks.Read`) permission, or it is an application-only token.
- **Solution:** Consent to `Tasks.ReadWrite` and use a delegated (signed-in user) token.

#### `Microsoft To Do error: ... ResourceNotFound` or `BadRequest`
- **Cause:** `listId` or `taskId` is empty, wrong, or the item was deleted.
- **Solution:** Copy the IDs from `getLists` / `getTasks`. IDs are long and may end with `=`; copy them completely.

#### `Microsoft To Do error` on `createTask`
- **Cause:** `title` is empty, `importance` is not `low`, `normal` or `high`, or `dueDateTime` is not a valid date.
- **Solution:** Fill in `title` and use the format `YYYY-MM-DDTHH:mm:ss` for the due date.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-09 | Full documentation: parameters, operations, authentication, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->
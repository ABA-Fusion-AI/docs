---
node_id: "microsoft-todo"
title: "Microsoft To Do"
description: "Manage task lists and tasks in Microsoft To Do via Microsoft Graph API."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-09"
author: "Fusion Team"
tags:
  - microsoft-todo
  - microsoft-graph
  - tasks
  - productivity
  - integration
  - peer-only
related_nodes:
  - ms-todo
  - http-request
  - function
---

<!-- SECTION: overview -->
# Microsoft To Do

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage task lists and tasks in [Microsoft To Do](https://to-do.office.com) through the Microsoft Graph API: list, read and create task lists, and list, read, create, update, complete and delete tasks. The node calls `https://graph.microsoft.com/v1.0/me/todo` with a Microsoft OAuth 2.0 access token, so it works on the To Do account of the signed-in user (personal Microsoft account or Microsoft 365 work account).

### Use Cases

- **Task creation from events:** Create a To Do task when a form is submitted, an e-mail arrives, or a ticket is opened.
- **Follow-up tracking:** List the tasks of a list and act on the ones that are not completed yet.
- **Workflow closure:** Mark a task as completed when the related process finishes.
- **List setup:** Create a dedicated task list for a project or a team.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `listTaskLists` | Operation to perform (see below). |
| `accessToken` | `string` | Yes | — | Microsoft Graph OAuth 2.0 access token with the `Tasks.ReadWrite` permission. Supports expressions. |
| `listId` | `string` | For all operations except `listTaskLists` and `createTaskList` | — | Task list ID. Supports expressions. |
| `taskId` | `string` | For `getTask`, `updateTask`, `completeTask`, `deleteTask` | — | Task ID. Supports expressions. |
| `displayName` | `string` | For `createTaskList` | — | Name of the new task list. Supports expressions. |
| `title` | `string` | For `createTask` | — | Task title (also used by `updateTask`). Supports expressions. |
| `bodyContent` | `string` | No | — | Task notes, plain text. Used by `createTask` only. Supports expressions. |
| `importance` | `string` | No | — | `low`, `normal`, or `high`. Used by `createTask` and `updateTask`. Supports expressions. |
| `dueDateTime` | `string` | No | — | Due date, ISO format without time zone (e.g. `2026-12-31T00:00:00`), interpreted as UTC. Used by `createTask` only. Supports expressions. |
| `limit` | `string` | No | `20` | Maximum number of tasks returned by `listTasks`. Supports expressions. |

### Available Operations

| Operation | Description | Graph request |
|-----------|-------------|---------------|
| `listTaskLists` | List all task lists. | `GET /me/todo/lists` |
| `getTaskList` | Get one task list. | `GET /me/todo/lists/{listId}` |
| `createTaskList` | Create a task list named `displayName`. | `POST /me/todo/lists` |
| `listTasks` | List the tasks of a list (up to `limit`). | `GET /me/todo/lists/{listId}/tasks?$top={limit}` |
| `getTask` | Get one task. | `GET /me/todo/lists/{listId}/tasks/{taskId}` |
| `createTask` | Create a task with `title`, `bodyContent`, `importance`, `dueDateTime`. | `POST /me/todo/lists/{listId}/tasks` |
| `updateTask` | Change the `title` and/or `importance` of a task. | `PATCH /me/todo/lists/{listId}/tasks/{taskId}` |
| `completeTask` | Set the task status to `completed`. | `PATCH /me/todo/lists/{listId}/tasks/{taskId}` |
| `deleteTask` | Delete a task. | `DELETE /me/todo/lists/{listId}/tasks/{taskId}` |

> `updateTask` only updates `title` and `importance`; `bodyContent` and `dueDateTime` are ignored. To change the notes or the due date, use the `ms-todo` node. This node has no operation to delete a task list.

All parameters are shown for every operation. The node checks the required IDs and returns a clear error (e.g. `listId and taskId are required for getTask`) before calling the API.

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
| `success` | `object` | The Microsoft Graph response (see examples). `deleteTask` returns `{}`. |
| `error` | `Error` | A missing-parameter error, or `Microsoft To Do Error <status>: <response>`, e.g. `Microsoft To Do Error 401: {"error":{"code":"InvalidAuthenticationToken",...}}`. |

### Output Examples

#### `listTaskLists`

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

#### `createTask`

```json
{
  "importance": "high",
  "status": "notStarted",
  "title": "Prepare the monthly report",
  "createdDateTime": "2026-10-09T10:21:05.1234567Z",
  "lastModifiedDateTime": "2026-10-09T10:21:05.1234567Z",
  "id": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgBGAAAD...",
  "body": {
    "content": "Created by Fusion",
    "contentType": "text"
  },
  "dueDateTime": {
    "dateTime": "2026-12-31T00:00:00.0000000",
    "timeZone": "UTC"
  }
}
```

#### `listTasks`

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('user%40example.com')/todo/lists('...')/tasks",
  "value": [
    {
      "importance": "high",
      "status": "notStarted",
      "title": "Prepare the monthly report",
      "id": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgBGAAAD..."
    }
  ]
}
```

`completeTask` and `updateTask` return the full updated task (e.g. `"status": "completed"`).

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
2. **Microsoft To Do Node:** Runs the configured operation (e.g. `listTaskLists`).
3. **Log Node:** Displays the Microsoft Graph response.

### Example Configurations

#### Create a task

```json
{
  "operation": "createTask",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "listId": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgAuAAAD...",
  "title": "Call back {{ input.customerName }}",
  "bodyContent": "Request received from {{ input.email }}",
  "importance": "high",
  "dueDateTime": "2026-12-31T00:00:00"
}
```

#### Complete a task

```json
{
  "operation": "completeTask",
  "accessToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "listId": "AQMkADAwATM0MDAAMS0wMjVlLTcyMWMtMDACLTAwCgAuAAAD...",
  "taskId": "{{ input.taskId }}"
}
```

### Common Patterns

- **Webhook → Microsoft To Do (`createTask`):** Turn incoming requests into tasks.
- **Cron → Microsoft To Do (`listTasks`) → Function → Notification:** Send a daily summary of open tasks.
- **Microsoft To Do (`listTaskLists`) → Function:** Find a list ID by its `displayName` before creating tasks in it.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `accessToken` in Fusion's **Secrets** system. Never paste it directly into workflow parameters or commit it to version control.

- Request only the permissions the workflow needs (`Tasks.Read` for read-only use).
- Access tokens are short-lived; tokens copied from Graph Explorer are meant for testing only.
- Task titles and notes can contain personal data; avoid logging full responses in shared workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Microsoft To Do Error 401: ... InvalidAuthenticationToken`
- **Cause:** The access token is missing, malformed, or expired (tokens usually last about 1 hour).
- **Solution:** Get a new token and update `accessToken`.

#### `Microsoft To Do Error 403: ... Forbidden`
- **Cause:** The token does not include the `Tasks.ReadWrite` (or `Tasks.Read`) permission, or it is an application-only token.
- **Solution:** Consent to `Tasks.ReadWrite` and use a delegated (signed-in user) token.

#### `Microsoft To Do Error 404: ... ResourceNotFound`
- **Cause:** `listId` or `taskId` is wrong, or the item was deleted.
- **Solution:** Copy the IDs from `listTaskLists` / `listTasks`. IDs are long and may end with `=`; copy them completely.

#### `Microsoft To Do Error 400` on `createTask`
- **Cause:** `importance` is not `low`, `normal` or `high`, or `dueDateTime` is not a valid date.
- **Solution:** Use one of the allowed values and the format `YYYY-MM-DDTHH:mm:ss`.

#### `updateTask` does not change the notes or the due date
- **Cause:** This node only sends `title` and `importance` on update.
- **Solution:** Use the `ms-todo` node's `updateTask`, which also updates `body` and `dueDateTime`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-09 | Full documentation: parameters, operations, authentication, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->
---
node_id: "taiga"
title: "Taiga"
description: "Manage Taiga agile projects: list projects and members, and create, read, update and delete issues, user stories and sprints (milestones)."
category: "peer-only"
subcategory: "Business & Productivity"
version: "1.0.0"
language: "en"
last_updated: "2026-11-23"
author: "Fusion Team"
tags:
  - taiga
  - agile
  - scrum
  - project-management
  - issues
  - user-stories
  - sprints
related_nodes:
  - trello
  - linear
---

<!-- SECTION: overview -->
# Taiga

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node &nbsp;&nbsp;|&nbsp;&nbsp;**AI Tool:** Yes

Automate agile project management in **Taiga**: read **projects** and **members**, and manage **issues**, **user stories** and **sprints (milestones)**. The node signs in once with your Taiga username and password when the workflow starts, then uses the returned token for every call. It offers **18 operations**:

| Resource | Operations |
|----------|------------|
| Projects | `listProjects`, `getProject` |
| Members | `listMembers` |
| Issues | `listIssues`, `getIssue`, `createIssue`, `updateIssue`, `deleteIssue` |
| User stories | `listUserStories`, `getUserStory`, `createUserStory`, `updateUserStory`, `deleteUserStory` |
| Milestones (sprints) | `listMilestones`, `getMilestone`, `createMilestone`, `updateMilestone`, `deleteMilestone` |

### Use Cases

- **Bug intake:** Create a Taiga issue from a form, an email or a monitoring alert.
- **Backlog sync:** Create user stories from another tool (spreadsheet, CRM, support desk).
- **Sprint planning:** Create the next sprint automatically and assign stories to it.
- **Reporting:** List open issues or the stories of a sprint and send a summary to a chat.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | `string` | Yes | Taiga username or email. |
| `password` | `string` | Yes | Taiga password. |
| `baseUrl` | `string` | Yes | Taiga API URL **including `/api/v1`**. Cloud: `https://api.taiga.io/api/v1`. Self-hosted (Docker): `http://<host>:9000/api/v1`. |
| `operation` | `enum` | Yes | Operation to perform (default `listProjects`). |

> The node signs in with `POST {baseUrl}/auth` when the workflow starts. If `baseUrl` does not end with `/api/v1`, Taiga returns a web page instead of JSON and the workflow stays in **Setup failed** (see Troubleshooting).

### Identifiers

| Parameter | Used by | Description |
|-----------|---------|-------------|
| `projectId` | `getProject`, `listMembers`, list operations, `createIssue`, `createUserStory`, `createMilestone` | Numeric project ID, from `listProjects`. |
| `issueId` | `getIssue`, `updateIssue`, `deleteIssue` | Numeric issue ID (not the `#ref` shown in the UI). |
| `storyId` | `getUserStory`, `updateUserStory`, `deleteUserStory` | Numeric user story ID. |
| `milestoneId` | `getMilestone`, `updateMilestone`, `deleteMilestone` | Numeric sprint ID, from `listMilestones`. |

> Taiga objects have an internal `id` and a project-level `ref` (the `#3` in the UI). The node always uses the **`id`**.

### List filters (`listIssues`, `listUserStories`)

| Filter | Description |
|--------|-------------|
| `page` / `pageSize` | Pagination (page size 1–100, default 30). |
| `statusId` | Filter by status ID. |
| `assignedToId` | Filter by assignee (the `user` field of `listMembers`). |
| `milestoneId` | Filter by sprint. |
| `tags` | Comma-separated tags, e.g. `bug,urgent`. |
| `q` (Search Query) | Free-text search in subject and description. |

### Issue data (`createIssue`, `updateIssue`)

| Field | Description |
|-------|-------------|
| `subject` | Title — **required** for `createIssue`. |
| `description` | Text of the issue. |
| `statusId`, `priorityId`, `severityId`, `typeId` | Numeric IDs of the project's statuses, priorities, severities and types (Taiga defaults are used if empty). |
| `assignedToId` | Assignee user ID. |
| `milestoneId` | Sprint ID. |
| `tagsJson` | JSON array of tags, e.g. `["bug","urgent"]`. |
| `dueDate` | `YYYY-MM-DD`. |
| `version` | **Required for `updateIssue`**: the current `version` of the issue (from `getIssue` or the create response). |

### User story data (`createUserStory`, `updateUserStory`)

Same as issue data without priority, severity and type: `subject` (required for create), `description`, `statusId`, `assignedToId`, `milestoneId`, `tagsJson`, `dueDate`, and `version` (**required for `updateUserStory`**).

### Milestone data (`createMilestone`, `updateMilestone`)

| Field | Description |
|-------|-------------|
| `name` | Sprint name — **required** for create; must be unique in the project. |
| `estimatedStart` | `YYYY-MM-DD` — **required** for create. |
| `estimatedFinish` | `YYYY-MM-DD` — **required** for create. |
| `closed` | `updateMilestone` only: `true` to close the sprint. |

> Taiga uses **optimistic locking**: every update must send the object's current `version`. After a successful update, the version increases by 1 (e.g. 1 → 2).

### Taiga account

- **Taiga Cloud:** create a free account on taiga.io and a project (Scrum template for sprints). Use `https://api.taiga.io/api/v1`.
- **Self-hosted:** use your server URL with `/api/v1`.
- The user must be a member of the project; `listProjects` only returns the projects of the signed-in user.

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
| `success` | `object` / `array` | Taiga object(s). Delete operations return `{ "success": true, "<type>Id": <id> }`. |
| `error` | `Error` | Missing fields or API errors (`Taiga operation failed: Taiga API error (<status>): ...`). |

### Output Examples

#### `listProjects` (excerpt)

```json
[
  {
    "id": 1805817,
    "name": "testfusion",
    "slug": "yassinekaram-testfusion",
    "owner": { "id": 959447, "username": "yassinekaram" },
    "members": [959447]
  }
]
```

#### `listMembers` (excerpt)

```json
[
  { "id": 3062792, "user": 959447, "project": 1805817, "role_name": "Product Owner", "is_admin": true }
]
```

#### `createIssue` / `updateIssue` (excerpt)

```json
{
  "id": 2404012,
  "ref": 3,
  "subject": "Fusion node test issue (updated)",
  "project": 1805817,
  "status_extra_info": { "name": "New", "is_closed": false },
  "version": 2
}
```

#### `createMilestone` (excerpt)

```json
{
  "id": 542482,
  "name": "Fusion test sprint",
  "estimated_start": "2026-11-23",
  "estimated_finish": "2026-12-07",
  "closed": false
}
```

#### Delete operations

```json
{ "success": true, "issueId": 2404012 }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Taiga in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Taiga Node:** Signs in, then runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Create an issue from an alert

```json
{
  "username": "{{ secrets.TAIGA_USER }}",
  "password": "{{ secrets.TAIGA_PASSWORD }}",
  "baseUrl": "https://api.taiga.io/api/v1",
  "operation": "createIssue",
  "projectId": 1805817,
  "issueData": {
    "subject": "Alert: {{ input.title }}",
    "description": "{{ input.message }}",
    "tagsJson": "[\"alert\",\"auto\"]"
  }
}
```

#### Update a user story

```json
{
  "operation": "updateUserStory",
  "storyId": 9732194,
  "storyData": { "subject": "Checkout page v2", "version": 1 }
}
```

#### Create a sprint

```json
{
  "operation": "createMilestone",
  "projectId": 1805817,
  "milestoneData": { "name": "Sprint 12", "estimatedStart": "2026-11-23", "estimatedFinish": "2026-12-07" }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `username` and `password` in Fusion's **Secrets** vault. Do not commit them into version control.

- Use a dedicated Taiga account with access only to the projects the workflow needs.
- Delete operations are permanent: protect them from repeated triggers.
- The node is available as an **AI tool**: limit the operations exposed to an agent if it should not delete data.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Workflow stays in **Setup failed** with `Unexpected token '<', "<!doctype "... is not valid JSON`
- **Cause:** `baseUrl` does not end with `/api/v1` (e.g. `https://api.taiga.io`), so the sign-in request gets an HTML page.
- **Solution:** Use `https://api.taiga.io/api/v1` (or `http://<host>:9000/api/v1`), then click **Cancel retry** and start the workflow again.

#### `Taiga auth error (401)`
- **Cause:** Wrong username or password.
- **Solution:** Check the credentials (username or email).

#### `Taiga API error (400)` on `updateIssue` / `updateUserStory`
- **Cause:** `version` is missing or outdated (someone changed the object).
- **Solution:** Read the object again with `get…` and send its current `version`.

#### `Taiga API error (400): {"name":["Duplicated name"]}`
- **Cause:** A sprint with the same name already exists in the project.
- **Solution:** Use a unique sprint name.

#### `Taiga API error (404)`
- **Cause:** Wrong ID, or a `ref` (`#3`) was used instead of the `id`.
- **Solution:** Get the `id` from the list operations.

#### `listProjects` returns an empty list
- **Cause:** The signed-in user is not a member of any project.
- **Solution:** Add the user to the project.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-23 | Initial documentation: 18 operations, connection, filters, data fields, output examples, troubleshooting |

<!-- /SECTION: changelog -->

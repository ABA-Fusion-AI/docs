---
node_id: "linear"
title: "Linear"
description: "Manage Linear issues, teams, projects, users and workflow states through the Linear GraphQL API."
category: "integrations"
subcategory: "project-management"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - linear
  - issues
  - project-management
  - graphql
  - engineering
related_nodes:
  - linear-trigger
  - jira
  - trello
  - mondaycom
---

<!-- SECTION: overview -->
# Linear

> **Category:** Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Automate **Linear** from a Fusion workflow with the Linear GraphQL API (`https://api.linear.app/graphql`). The node offers **11 operations**:

| Resource | Operations |
|----------|------------|
| Issues | `getIssues`, `getIssue`, `createIssue`, `updateIssue`, `deleteIssue` |
| Teams | `getTeams`, `getTeam` |
| Projects | `getProjects` |
| Users | `getCurrentUser`, `getUsers` |
| Workflow states | `getStates` |

### Use Cases

- **Bug intake:** Create a Linear issue from an error alert, a support ticket or a form.
- **Status sync:** Move an issue to *Done* when a deployment or a pull request finishes.
- **Triage:** Assign issues and set their priority automatically.
- **Reporting:** List issues, projects and users to build a weekly summary.

To react to events in Linear (issue created, updated…), use the **Linear Trigger** node.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | Linear personal API key. Supports expressions. |
| `operation` | `enum` | Yes | `getIssues` | Operation to perform (see below). |
| `issueId` | `string` | For `getIssue`, `updateIssue`, `deleteIssue` | — | Issue UUID or identifier (e.g. `FUS-6`). |
| `teamId` | `string` | For `getTeam` (optional for `getStates`) | — | Team UUID. |
| `issueFilters` | `object` | No | — | Filters for `getIssues` (see below). |
| `issueData` | `object` | For `createIssue`, `updateIssue` | — | Issue fields (see below). |
| `projectFilters` | `object` | No | — | `first`, `after`, `filter` for `getProjects`. |
| `userFilters` | `object` | No | — | `first`, `after` for `getUsers`. |

### `issueFilters` (getIssues)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `first` | `number` (1–100) | `50` | Number of issues per page. |
| `after` | `string` | — | Cursor of the next page (`pageInfo.endCursor` of the previous call). |
| `orderBy` | `string` | `updatedAt` | `createdAt` or `updatedAt`. |
| `filter` | `object` | — | Linear `IssueFilter`, e.g. `{ "team": { "key": { "eq": "FUS" } }, "state": { "type": { "eq": "started" } } }`. |

### `issueData` (createIssue / updateIssue)

| Field | Type | createIssue | Description |
|-------|------|-------------|-------------|
| `teamId` | `string` | **Required** | Team of the issue. |
| `title` | `string` | **Required** | Issue title. |
| `description` | `string` | Optional | Markdown description. |
| `assigneeId` | `string` | Optional | User ID of the assignee. |
| `stateId` | `string` | Optional | Workflow state ID (from `getStates`). Default: the team's first state (e.g. *Backlog*). |
| `priority` | `number` | Optional | `0` none, `1` urgent, `2` high, `3` medium, `4` low. |
| `projectId` | `string` | Optional | Project ID. |
| `labelIds` | `string[]` | Optional | Label IDs. |
| `dueDate` | `string` | Optional | Due date `YYYY-MM-DD` (a full ISO date is cut to the day). |
| `estimate` | `number` | Optional | Estimate in points (whole number). |

For `updateIssue`, only the fields you fill in are changed; at least one field is required.

### Authentication & API Key

1. In Linear, open **Settings → Account → Security & access** (or **API**).
2. Under **Personal API keys**, create a new key.
3. Paste it into the `apiKey` field (or store it in Fusion Secrets). The key has the same rights as your user.

### Where to find the IDs

| ID | How to get it |
|----|---------------|
| `teamId` | `getTeams` |
| `stateId` | `getStates` (with `teamId`) or `getTeam` |
| `assigneeId` | `getUsers` or `getCurrentUser` |
| `projectId` | `getProjects` |
| `issueId` | `getIssues`, `createIssue`, or the identifier shown in Linear (e.g. `FUS-6`) |

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
| `success` | `object` | GraphQL response of Linear (`{ "data": { ... } }`). `deleteIssue` returns `{ "success": true }`. |
| `error` | `Error` | Emitted on missing fields, invalid values or API errors. |

### Output Examples

#### `createIssue`

```json
{
  "data": {
    "issueCreate": {
      "success": true,
      "issue": {
        "id": "736a1228-0dd4-4831-82ef-9fb5dcab6913",
        "identifier": "FUS-6",
        "title": "Issue de test Fusion",
        "description": null,
        "priority": 2,
        "state": { "id": "9b8f92e8-f062-4dff-ab54-fb5f8f793515", "name": "Backlog" },
        "assignee": { "id": "70f19178-46a7-448c-95a4-bfcc91b37e5f", "name": "Karam Yassine" },
        "team": { "id": "dced20e5-7a0a-4b35-9536-e2eedb329986", "name": "FusionTest02" }
      }
    }
  }
}
```

#### `getTeams`

```json
{
  "data": {
    "teams": {
      "nodes": [
        {
          "id": "dced20e5-7a0a-4b35-9536-e2eedb329986",
          "name": "FusionTest02",
          "key": "FUS",
          "description": null,
          "private": false,
          "members": { "nodes": [ { "id": "70f19178-...", "name": "Karam Yassine", "email": "..." } ] }
        }
      ]
    }
  }
}
```

#### `deleteIssue`

```json
{ "success": true }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Create a Linear issue
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Linear Node:** Runs the selected operation.
3. **Log Node:** Displays the Linear response.

A typical chain: **getTeams → getStates → createIssue → updateIssue**, passing the IDs with expressions.

### Example Configuration

```json
{
  "apiKey": "{{ secrets.LINEAR_API_KEY }}",
  "operation": "createIssue",
  "issueData": {
    "teamId": "dced20e5-7a0a-4b35-9536-e2eedb329986",
    "title": "Error on checkout: {{ input.error }}",
    "description": "Reported automatically by Fusion.",
    "priority": 2,
    "dueDate": "2026-12-01"
  }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Linear `apiKey` in Fusion's **Secrets** vault. Do not commit it into version control.

- A personal API key has the same rights as your Linear user: use a dedicated user for automations if possible.
- `deleteIssue` is permanent from the workflow's point of view: restrict who can run it.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Linear operation failed: Team ID and title are required`
- **Cause:** `issueData.teamId` or `issueData.title` is empty for `createIssue`.
- **Solution:** Fill in both (get the team ID with `getTeams`).

#### `Linear operation failed: Issue ID is required` / `Team ID is required`
- **Cause:** The selected operation needs an ID that is empty.
- **Solution:** Fill in `issueId` or `teamId`.

#### `Nothing to update: provide at least one field in issueData.`
- **Cause:** `updateIssue` was run without any field to change.
- **Solution:** Fill in at least one `issueData` field.

#### `Invalid dueDate '...': use the YYYY-MM-DD format.` / `Invalid estimate '...'`
- **Cause:** The due date is not a date, or the estimate is not a whole number.
- **Solution:** Use `2026-12-01` and integer estimates (`1`, `2`, `3`…).

#### `Invalid orderBy '...': use 'createdAt' or 'updatedAt'.`
- **Cause:** Unsupported sort field in `issueFilters.orderBy`.
- **Solution:** Use `createdAt` or `updatedAt`.

#### `Linear API error (401)` / `Authentication required`
- **Cause:** Invalid or revoked API key.
- **Solution:** Create a new personal API key in Linear.

#### `Linear API error: Entity not found` / `Argument Validation Error`
- **Cause:** Wrong ID (issue, team, state, user) or an ID from another team.
- **Solution:** Get the IDs again with the `get…` operations.

#### `createIssue` takes several seconds
- **Cause:** Normal: the creation call is slower than read operations (around 8–10 s).
- **Solution:** No action needed.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: 11 operations, filters, issue fields, authentication, output examples, troubleshooting |

<!-- /SECTION: changelog -->

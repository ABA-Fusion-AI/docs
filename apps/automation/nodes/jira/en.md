---
node_id: "jira-action"
title: "Jira Action"
description: "Interact with Jira — manage issues, comments, transitions, sprints, boards, projects, and users via the Jira REST API"
category: "integrations"
subcategory: "project-management"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - jira
  - project-management
  - issues
  - sprints
  - agile
  - action
related_nodes:
  - slack-action
  - jenkins-action
  - webhook-trigger
  - function
  - cron
---

<!-- SECTION: header -->
# Jira Action

> **Category:** Integrations | **Type:** Action Node

Connect to a Jira instance and manage the full project lifecycle — create and update issues, post comments, trigger transitions, manage sprints and boards, link issues, and query users.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Jira Action** node authenticates against the Jira REST API using Basic Auth (email + API token) and executes one of 21 available operations. It covers the complete Jira workflow: issue CRUD, comments, transitions, assignments, project metadata, issue linking, board and sprint management, and user lookup.

### Key Features

- **21 Operations:** Full coverage of issues, comments, transitions, assignments, projects, issue links, boards, sprints, and users
- **Issue Creation:** Create richly structured issues with type, priority, labels, components, and parent (for subtasks)
- **Workflow Transitions:** Move issues through workflow states by transition name or ID
- **Sprint Management:** List sprints, inspect sprint issues, and move issues between sprints
- **Board Support:** List boards and retrieve their backlog or active issues
- **User Lookup:** Search users by query string or fetch by exact account ID
- **Pagination:** `maxResults` and `startAt` parameters on list operations for full result control
- **Expression Support:** `issueKey`, `projectKey`, `boardId`, `sprintId`, and `issueKeys` accept dynamic expressions from upstream nodes

### Use Cases

- Automatically create a Jira issue when a critical alert fires in a monitoring workflow
- Transition an issue to "In Review" after a Jenkins build succeeds
- Post a comment on a ticket with the result of an automated test run
- Move all issues from one sprint to the next at the end of a sprint cycle
- Assign an issue to the on-call engineer resolved from an upstream function node
- Notify a Slack channel with the link and status of a newly created issue

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | `string` | ✅ Yes | Jira instance URL (e.g. `https://your-org.atlassian.net`) |
| `email` | `string` | ✅ Yes | Jira account email address |
| `apiToken` | `string` | ✅ Yes | Jira API token — generate at **Account Settings → Security → API tokens** |

### Operation Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | — | Operation to execute (see below) |
| `issueKey` | `string` | Conditional | — | Jira issue key (e.g. `PROJECT-123`) — required for issue-specific operations |
| `projectKey` | `string` | Conditional | — | Project key (e.g. `PROJECT`) — required for project and issue-creation operations |
| `issueData` | `object` | Conditional | — | Issue fields for `createIssue` and `updateIssue` (see structure below) |
| `commentBody` | `string` | Conditional | — | Comment text — required for `addComment` |
| `transitionId` | `string` | Conditional | — | ID of the target transition — used by `transitionIssue` |
| `transitionName` | `string` | Conditional | — | Name of the target transition (resolved if `transitionId` is not provided) |
| `assigneeAccountId` | `string` | Conditional | — | Account ID of the assignee — required for `assignIssue` |
| `linkType` | `enum` | Conditional | — | Relationship type for `linkIssues`: `Relates`, `Blocks`, `Cloners`, `Duplicate` |
| `inwardIssue` | `string` | Conditional | — | Source issue key for `linkIssues` |
| `outwardIssue` | `string` | Conditional | — | Target issue key for `linkIssues` |
| `boardId` | `number` | Conditional | — | Board ID — required for `getBoardIssues`, `listSprints`, `getSprintIssues` |
| `sprintId` | `number` | Conditional | — | Sprint ID — required for `getSprintIssues` and `moveIssuesToSprint` |
| `issueKeys` | `string[]` | Conditional | — | Array of issue keys to move — required for `moveIssuesToSprint` |
| `query` | `string` | Conditional | — | Search string — required for `searchUsers` |
| `accountId` | `string` | Conditional | — | Exact account ID — required for `getUserByAccountId` |
| `maxResults` | `number` | ❌ No | `50` | Maximum number of results to return (1–100) |
| `startAt` | `number` | ❌ No | `0` | Zero-based offset for paginated list operations |

### `issueData` Structure

Used by `createIssue` and `updateIssue`:

```json
{
  "summary": "Fix login page crash on Safari",
  "description": "Users on Safari 17 experience a white screen after submitting the login form.",
  "issuetype": { "name": "Bug" },
  "priority": { "name": "High" },
  "assignee": { "accountId": "5b10a2844c20165700ede21g" },
  "labels": ["frontend", "safari"],
  "components": [{ "name": "Auth Service" }],
  "parent": { "key": "PROJECT-100" }
}
```

> `project.key` is set automatically from the `projectKey` parameter. Fields not provided in `updateIssue` are left unchanged.

### Available Operations

#### Issues

| Operation | Description |
|-----------|-------------|
| `createIssue` | Create a new issue in the specified project |
| `updateIssue` | Update fields of an existing issue |
| `deleteIssue` | Permanently delete an issue |
| `getIssue` | Retrieve full issue details by key |

#### Comments

| Operation | Description |
|-----------|-------------|
| `addComment` | Add a comment to an issue |
| `getComments` | List all comments on an issue |

#### Transitions

| Operation | Description |
|-----------|-------------|
| `transitionIssue` | Move an issue to a new workflow state |
| `getTransitions` | List all valid transitions for an issue |

#### Assignments

| Operation | Description |
|-----------|-------------|
| `assignIssue` | Assign an issue to a user by account ID |

#### Projects

| Operation | Description |
|-----------|-------------|
| `listProjects` | List all accessible projects |
| `getProject` | Retrieve project metadata by key |
| `getIssueTypesForProject` | List available issue types for a project |

#### Issue Links

| Operation | Description |
|-----------|-------------|
| `linkIssues` | Create a directional link between two issues |
| `getIssueLinks` | Retrieve all links attached to an issue |

#### Boards

| Operation | Description |
|-----------|-------------|
| `listBoards` | List all boards accessible to the authenticated user |
| `getBoardIssues` | Retrieve issues on a specific board |

#### Sprints

| Operation | Description |
|-----------|-------------|
| `listSprints` | List all sprints for a board |
| `getSprintIssues` | Retrieve issues in a specific sprint |
| `moveIssuesToSprint` | Move one or more issues to a sprint |

#### Users

| Operation | Description |
|-----------|-------------|
| `getCurrentUser` | Retrieve the profile of the authenticated user |
| `searchUsers` | Search for users by display name or email |
| `getUserByAccountId` | Retrieve a user profile by exact account ID |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — `issueKey`, `projectKey`, `boardId`, `sprintId`, and `issueKeys` can also be resolved from here via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the Jira API call |
| `error` | `Error` | Emitted if the operation or connection fails |

### Output Schemas by Operation

**`createIssue`**
```json
{
  "id": "10042",
  "key": "PROJECT-42",
  "self": "https://your-org.atlassian.net/rest/api/3/issue/10042"
}
```

**`getIssue`**
```json
{
  "id": "10042",
  "key": "PROJECT-42",
  "fields": {
    "summary": "Fix login page crash on Safari",
    "status": { "name": "In Progress" },
    "assignee": { "displayName": "Jane Doe", "accountId": "..." },
    "priority": { "name": "High" },
    "issuetype": { "name": "Bug" },
    "created": "2026-03-10T09:00:00.000+0000",
    "updated": "2026-03-10T14:30:00.000+0000"
  }
}
```

**`addComment`**
```json
{
  "id": "20001",
  "body": "Build #42 passed all checks.",
  "author": { "displayName": "Fusion Bot", "accountId": "..." },
  "created": "2026-03-10T15:00:00.000+0000"
}
```

**`getTransitions`**
```json
{
  "transitions": [
    { "id": "11", "name": "To Do" },
    { "id": "21", "name": "In Progress" },
    { "id": "31", "name": "In Review" },
    { "id": "41", "name": "Done" }
  ]
}
```

**`listProjects`**
```json
[
  { "id": "10000", "key": "PROJECT", "name": "My Project", "projectTypeKey": "software" },
  { "id": "10001", "key": "OPS", "name": "Operations", "projectTypeKey": "service_desk" }
]
```

**`listBoards`**
```json
{
  "values": [
    { "id": 1, "name": "Project Board", "type": "scrum" },
    { "id": 2, "name": "Ops Kanban", "type": "kanban" }
  ]
}
```

**`listSprints`**
```json
{
  "values": [
    { "id": 10, "name": "Sprint 1", "state": "closed" },
    { "id": 11, "name": "Sprint 2", "state": "active" },
    { "id": 12, "name": "Sprint 3", "state": "future" }
  ]
}
```

**`moveIssuesToSprint`**
```json
{
  "success": true,
  "sprintId": 12,
  "issueKeys": ["PROJECT-45", "PROJECT-46"]
}
```

**`searchUsers`**
```json
[
  {
    "accountId": "5b10a2844c20165700ede21g",
    "displayName": "Jane Doe",
    "emailAddress": "jane.doe@example.com",
    "active": true
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Create a Bug Issue

**Configuration:**
```json
{
  "host": "https://your-org.atlassian.net",
  "email": "bot@example.com",
  "apiToken": "your-api-token",
  "operation": "createIssue",
  "projectKey": "PROJECT",
  "issueData": {
    "summary": "API returns 500 on empty payload",
    "description": "Calling POST /api/data with an empty body triggers an unhandled exception.",
    "issuetype": { "name": "Bug" },
    "priority": { "name": "High" },
    "labels": ["backend", "api"]
  }
}
```

**Output:**
```json
{
  "id": "10055",
  "key": "PROJECT-55",
  "self": "https://your-org.atlassian.net/rest/api/3/issue/10055"
}
```

---

### Example: Transition an Issue to "Done"

Move `PROJECT-42` to the "Done" state using the transition name.

**Configuration:**
```json
{
  "host": "https://your-org.atlassian.net",
  "email": "bot@example.com",
  "apiToken": "your-api-token",
  "operation": "transitionIssue",
  "issueKey": "PROJECT-42",
  "transitionName": "Done"
}
```

> If both `transitionId` and `transitionName` are provided, `transitionId` takes precedence. Use `getTransitions` first to discover valid transition IDs.

---

### Example: Add a Comment with Build Results

Post a build summary comment automatically after a Jenkins job completes.

**Configuration:**
```json
{
  "host": "https://your-org.atlassian.net",
  "email": "bot@example.com",
  "apiToken": "your-api-token",
  "operation": "addComment",
  "issueKey": "{{input.issueKey}}",
  "commentBody": "✅ Build #{{input.buildNumber}} completed successfully. Duration: {{input.duration}}ms."
}
```

---

### Example: Move Issues to Next Sprint

**Configuration:**
```json
{
  "host": "https://your-org.atlassian.net",
  "email": "bot@example.com",
  "apiToken": "your-api-token",
  "operation": "moveIssuesToSprint",
  "sprintId": 12,
  "issueKeys": ["PROJECT-45", "PROJECT-46", "PROJECT-47"]
}
```

---

### Example: Search for a User to Assign

Find a user and then use their `accountId` in a subsequent `assignIssue` call.

**Configuration:**
```json
{
  "host": "https://your-org.atlassian.net",
  "email": "bot@example.com",
  "apiToken": "your-api-token",
  "operation": "searchUsers",
  "query": "jane.doe",
  "maxResults": 5
}
```

---

### Example: Create a Subtask

Create a subtask under a parent issue using the `parent` field.

**Configuration:**
```json
{
  "host": "https://your-org.atlassian.net",
  "email": "bot@example.com",
  "apiToken": "your-api-token",
  "operation": "createIssue",
  "projectKey": "PROJECT",
  "issueData": {
    "summary": "Write unit tests for login endpoint",
    "issuetype": { "name": "Subtask" },
    "parent": { "key": "PROJECT-42" }
  }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Jenkins Build → Jira Transition → Slack Notification

When a Jenkins build succeeds, transition the associated Jira issue to "In Review" and post a Slack notification.

```json
{
  "nodes": [
    {
      "id": "jenkins-webhook",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "check-success",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "if (input.result !== 'SUCCESS') return null; return input;"
      }
    },
    {
      "id": "transition-issue",
      "type": "jira-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "host": "https://your-org.atlassian.net",
        "email": "bot@example.com",
        "apiToken": "your-api-token",
        "operation": "transitionIssue",
        "issueKey": "{{input.issueKey}}",
        "transitionName": "In Review"
      }
    },
    {
      "id": "add-comment",
      "type": "jira-action",
      "position": { "x": 500, "y": 220 },
      "config": {
        "host": "https://your-org.atlassian.net",
        "email": "bot@example.com",
        "apiToken": "your-api-token",
        "operation": "addComment",
        "issueKey": "{{input.issueKey}}",
        "commentBody": "✅ Build #{{input.buildNumber}} passed. Moved to In Review."
      }
    },
    {
      "id": "notify-slack",
      "type": "slack-action",
      "position": { "x": 700, "y": 160 },
      "config": {
        "token": "xoxb-your-bot-token",
        "operation": "message",
        "messageOperation": "send",
        "messageChannelId": "C08XXXXXXXX",
        "text": "🔁 *{{input.issueKey}}* moved to *In Review* after build #{{input.buildNumber}}."
      }
    }
  ]
}
```

### Common Patterns

- **Bug Triage:** Webhook → Function (parse alert) → Jira `createIssue` → Jira `assignIssue` → Slack `send`
- **CI Gate:** Jenkins `getBuildInfo` → Function (check result) → Jira `transitionIssue` (pass/fail state)
- **Sprint Rollover:** Cron (end of sprint) → Jira `listSprints` → Function (find incomplete issues) → Jira `moveIssuesToSprint`
- **Log Attachment:** Jenkins `getBuildConsole` → Function (grep errors) → Jira `addComment` with error excerpt
- **Issue Linking:** Webhook (incident alert) → Jira `createIssue` → Jira `linkIssues` (links to parent epic)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to initialize Jira client`

**Cause:** `host` is unreachable, or credentials are invalid.

**Solution:** Verify the full URL including the `https://` scheme and no trailing slash. Ensure `email` matches the Atlassian account and `apiToken` is a valid API token (not your account password). Test by opening `https://your-org.atlassian.net/rest/api/3/myself` in a browser while logged in.

#### `issueKey is required`

**Cause:** An operation that requires an issue key (e.g. `getIssue`, `addComment`, `transitionIssue`) was called without providing one.

**Solution:** Provide the full issue key including the project prefix (e.g. `PROJECT-42`). You can use an expression like `{{input.issueKey}}` to resolve it from upstream data.

#### `Transition not found`

**Cause:** The `transitionName` provided does not match any valid transition for the issue in its current state.

**Solution:** Use `getTransitions` to list all available transitions for the issue and verify the exact name or use `transitionId` instead.

#### `boardId is required for listSprints`

**Cause:** `listSprints` or `getSprintIssues` was called without specifying a `boardId`.

**Solution:** Use `listBoards` first to discover available boards and their IDs, then pass the correct `boardId`.

#### `issueKeys must be an array`

**Cause:** `moveIssuesToSprint` received a string instead of an array for `issueKeys`.

**Solution:** Ensure `issueKeys` is an array of strings: `["PROJECT-1", "PROJECT-2"]`. If using an expression, wrap a single key in an array.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check `host` URL and network access |
| `401` | Unauthorized | Verify `email` and `apiToken` |
| `403` | Forbidden | User lacks permission for the operation or project |
| `404` | Issue / Project / Board not found | Check `issueKey`, `projectKey`, or `boardId` for typos |
| `400` | Bad request | Review `issueData` structure — required fields may be missing |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Slack Action](./slack-action.md) - Post issue notifications and build results to Slack channels
- [Jenkins Action](./jenkins.md) - Trigger builds and retrieve results to drive issue transitions
- [Cron](./cron.md) - Schedule sprint rollovers or recurring project health checks

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->

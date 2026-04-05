---
node_id: "gitlab-action"
title: "GitLab Action"
description: "Manage GitLab issues, merge requests, branches, pipelines, and projects via the GitLab REST API v4"
category: "integrations"
subcategory: "devops"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - gitlab
  - devops
  - issues
  - merge-requests
  - ci-cd
  - pipelines
  - branches
  - action
related_nodes:
  - github-action
  - jenkins-action
  - jira-action
  - discord-action
  - slack-action
---

<!-- SECTION: header -->
# GitLab Action

> **Category:** Integrations | **Type:** Action Node

Interact with GitLab repositories — manage issues, merge requests, branches, and CI/CD pipelines, and create projects, all via the GitLab REST API v4 with a Personal Access Token.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GitLab Action** node authenticates against the GitLab REST API v4 using a Personal Access Token (PAT) passed as the `PRIVATE-TOKEN` header. It supports 20 operations across five categories: issues, merge requests, pipelines (CI/CD), branches, and projects.

Project-scoped operations use a URL-encoded `owner/project` path as the project ID. Responses are normalized through a `formatResponse` layer for clean, minimal output.

### Key Features

- **20 Operations:** Full coverage of issues, MRs, pipelines, branches, and projects
- **Standard REST API:** Direct calls to `https://gitlab.com/api/v4` — works with any GitLab instance
- **Project Path Based:** Uses `owner/project` path instead of numeric project IDs — more readable
- **Pagination:** `page` and `per_page` on all list operations
- **Expression Support:** All key fields accept dynamic expressions from upstream nodes
- **Project Management:** `createProject` and `listProjects` for repository provisioning workflows

### Use Cases

- Create a GitLab issue automatically when a monitoring alert fires
- Trigger a CI/CD pipeline on a specific branch after a Jira sprint starts
- Open a merge request from a feature branch to `main` when a Jira issue moves to "In Review"
- Monitor running pipelines and cancel stuck ones as part of a maintenance workflow
- Provision a new GitLab project as part of a service onboarding automation

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | `string` | ✅ Yes | GitLab Personal Access Token with `api` scope |

> The token is sent as the `PRIVATE-TOKEN` header to `https://gitlab.com/api/v4`. Self-hosted instances are not yet configurable — the base URL is fixed to `https://gitlab.com`.

### Common Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to execute |
| `owner` | `string` | Conditional | GitLab username or group — required for all project-scoped operations |
| `project` | `string` | Conditional | Repository/project name — required for all project-scoped operations |
| `page` | `number` | ❌ No | Page number for list operations (default: 1) |
| `per_page` | `number` | ❌ No | Items per page for list operations (default: 20) |

> `owner` and `project` are combined into a URL-encoded path `owner%2Fproject` used as the GitLab project ID in API calls.

### Operation-Specific Parameters

| Parameter | Type | Required For | Description |
|-----------|------|-------------|-------------|
| `issue_iid` | `number` | `updateIssue`, `addComment` | Issue internal ID (the number shown in the UI) |
| `title` | `string` | `createIssue`, `createMergeRequest`, `createProject`; optional for updates | Title of the issue, MR, or project |
| `description` | `string` | Optional for issues and MRs | Body/description text |
| `comment` | `string` | `addComment` | Comment body text |
| `mr_source_branch` | `string` | `createMergeRequest` | Source branch (branch with changes) |
| `mr_target_branch` | `string` | `createMergeRequest` | Target branch (branch to merge into) |
| `mr_id` | `number` | `getMergeRequest`, `updateMergeRequest` | Merge request internal ID |
| `branch` | `string` | `createBranch`, `deleteBranch`, `getBranch` | Branch name |
| `source_branch` | `string` | `createBranch` | Branch to create from |
| `ref` | `string` | `triggerWorkflow` | Branch or tag to run the pipeline on |
| `pipeline_id` | `number` | `getWorkflow`, `cancelWorkflowRun` | Pipeline ID |
| `visibility` | `enum` | Optional for `createProject` | `private`, `public`, or `internal` (default: `private`) |

### Available Operations

#### Issues

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listIssues` | `owner`, `project` | List all issues in the project |
| `createIssue` | `owner`, `project`, `title` | Create a new issue |
| `updateIssue` | `owner`, `project`, `issue_iid` | Update issue title or description |
| `addComment` | `owner`, `project`, `issue_iid`, `comment` | Add a comment (note) to an issue |

#### Merge Requests

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `createMergeRequest` | `owner`, `project`, `title`, `mr_source_branch`, `mr_target_branch` | Open a new merge request |
| `listMergeRequests` | `owner`, `project` | List all merge requests |
| `getMergeRequest` | `owner`, `project`, `mr_id` | Get details of a specific MR |
| `updateMergeRequest` | `owner`, `project`, `mr_id` | Update MR title or description |

#### Pipelines (CI/CD)

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `triggerWorkflow` | `owner`, `project`, `ref` | Trigger a new pipeline on a branch or tag |
| `listWorkflows` | `owner`, `project` | List all pipelines in the project |
| `listWorkflowRuns` | `owner`, `project` | List currently running pipelines |
| `getWorkflow` | `owner`, `project`, `pipeline_id` | Get details of a specific pipeline |
| `cancelWorkflowRun` | `owner`, `project`, `pipeline_id` | Cancel a running pipeline |

#### Branches

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `createBranch` | `owner`, `project`, `branch`, `source_branch` | Create a new branch |
| `deleteBranch` | `owner`, `project`, `branch` | Delete a branch |
| `listBranches` | `owner`, `project` | List all branches |
| `getBranch` | `owner`, `project`, `branch` | Get details of a specific branch |

#### Projects

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `createProject` | `title` | Create a new GitLab project |
| `listProjects` | — | List all projects owned by the authenticated user |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream data — key fields can be resolved via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Formatted result of the operation |
| `error` | `Error` | Emitted if the API call fails or params are missing |

### Output Schemas

**`listIssues`**
```json
{
  "count": 5,
  "issues": [
    { "iid": 3, "title": "Fix sidebar bug", "state": "opened", "created_at": "2026-03-01T...", "url": "https://gitlab.com/...", "author": "driss" }
  ]
}
```

**`createIssue`**
```json
{
  "success": true,
  "iid": 4,
  "title": "New issue title",
  "url": "https://gitlab.com/my-group/my-project/-/issues/4",
  "created_at": "2026-03-11T..."
}
```

**`addComment`**
```json
{
  "success": true,
  "note_id": 12345678,
  "created_at": "2026-03-11T...",
  "author": "fusion-bot"
}
```

**`createMergeRequest`**
```json
{
  "success": true,
  "iid": 7,
  "title": "Feature: new auth flow",
  "source_branch": "feature/auth",
  "target_branch": "main",
  "url": "https://gitlab.com/my-group/my-project/-/merge_requests/7",
  "created_at": "2026-03-11T..."
}
```

**`getMergeRequest`**
```json
{
  "iid": 7,
  "title": "Feature: new auth flow",
  "state": "opened",
  "description": "This MR adds...",
  "source_branch": "feature/auth",
  "target_branch": "main",
  "url": "https://gitlab.com/...",
  "merge_status": "can_be_merged",
  "created_at": "2026-03-10T...",
  "updated_at": "2026-03-11T...",
  "author": "driss"
}
```

**`triggerWorkflow`**
```json
{
  "success": true,
  "id": 987654,
  "status": "pending",
  "ref": "main",
  "url": "https://gitlab.com/my-group/my-project/-/pipelines/987654"
}
```

**`getWorkflow`**
```json
{
  "id": 987654,
  "status": "success",
  "ref": "main",
  "sha": "a3f8c2d",
  "url": "https://gitlab.com/...",
  "created_at": "2026-03-11T...",
  "updated_at": "2026-03-11T..."
}
```

**`createBranch`**
```json
{
  "success": true,
  "name": "feature/new-feature",
  "commit_sha": "a3f8c2d",
  "url": "https://gitlab.com/my-group/my-project/-/tree/feature/new-feature"
}
```

**`listWorkflows`** (pipelines)
```json
{
  "count": 10,
  "pipelines": [
    { "id": 987654, "status": "success", "ref": "main", "url": "https://gitlab.com/...", "created_at": "2026-03-10T...", "updated_at": "2026-03-10T..." }
  ]
}
```

**`createProject`**
```json
{
  "success": true,
  "id": 12345,
  "name": "my-new-service",
  "path": "my-new-service",
  "visibility": "private",
  "url": "https://gitlab.com/my-group/my-new-service",
  "created_at": "2026-03-11T..."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Create an Issue from an Alert

**Configuration:**
```json
{
  "token": "glpat-your-token",
  "operation": "createIssue",
  "owner": "my-group",
  "project": "my-project",
  "title": "🚨 Alert: {{input.alertName}} on {{input.server}}",
  "description": "**Details:** {{input.details}}\n**Time:** {{input.triggeredAt}}"
}
```

---

### Example: Trigger a Pipeline on Main

**Configuration:**
```json
{
  "token": "glpat-your-token",
  "operation": "triggerWorkflow",
  "owner": "my-group",
  "project": "my-project",
  "ref": "main"
}
```

**Output:**
```json
{ "success": true, "id": 987654, "status": "pending", "ref": "main", "url": "..." }
```

---

### Example: Create a Feature Branch

**Configuration:**
```json
{
  "token": "glpat-your-token",
  "operation": "createBranch",
  "owner": "my-group",
  "project": "my-project",
  "branch": "feature/{{input.featureName}}",
  "source_branch": "develop"
}
```

---

### Example: Open a Merge Request

**Configuration:**
```json
{
  "token": "glpat-your-token",
  "operation": "createMergeRequest",
  "owner": "my-group",
  "project": "my-project",
  "title": "{{input.issueKey}}: {{input.summary}}",
  "mr_source_branch": "feature/{{input.issueKey}}",
  "mr_target_branch": "main",
  "description": "Linked to issue: {{input.issueKey}}"
}
```

---

### Example: Cancel a Stuck Pipeline

**Configuration:**
```json
{
  "token": "glpat-your-token",
  "operation": "cancelWorkflowRun",
  "owner": "my-group",
  "project": "my-project",
  "pipeline_id": "{{input.pipeline_id}}"
}
```

---

### Example: Create a New Project

**Configuration:**
```json
{
  "token": "glpat-your-token",
  "operation": "createProject",
  "title": "{{input.serviceName}}-service",
  "visibility": "private"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Jira Transition → GitLab Branch + MR + Discord

When a Jira issue moves to "In Review", create a GitLab branch, open an MR, and notify Discord.

```json
{
  "nodes": [
    {
      "id": "jira-webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "check-status",
      "type": "function",
      "config": {
        "code": "if (input.transition.to.name !== 'In Review') return null; return { issueKey: input.issue.key, summary: input.issue.fields.summary };"
      }
    },
    {
      "id": "create-branch",
      "type": "gitlab-action",
      "config": {
        "token": "glpat-your-token",
        "operation": "createBranch",
        "owner": "my-group",
        "project": "my-project",
        "branch": "review/{{input.issueKey}}",
        "source_branch": "develop"
      }
    },
    {
      "id": "create-mr",
      "type": "gitlab-action",
      "config": {
        "token": "glpat-your-token",
        "operation": "createMergeRequest",
        "owner": "my-group",
        "project": "my-project",
        "title": "{{input.issueKey}}: {{input.summary}}",
        "mr_source_branch": "review/{{input.issueKey}}",
        "mr_target_branch": "main"
      }
    },
    {
      "id": "notify-discord",
      "type": "discord-action",
      "config": {
        "operation": "sendEmbed",
        "channelId": "1234567890123456789",
        "embed": {
          "title": "🔀 MR created for {{input.issueKey}}",
          "description": "{{input.summary}}\n[View MR]({{input.url}})",
          "color": "purple"
        }
      }
    }
  ]
}
```

### Common Patterns

- **Alert → Issue:** Monitoring webhook → GitLab `createIssue` → Slack notify
- **Sprint Start → Branch:** Jira sprint event → GitLab `createBranch` (per issue)
- **Pipeline Monitor:** Cron → GitLab `listWorkflowRuns` → Function (find stuck) → `cancelWorkflowRun`
- **Onboarding Pipeline:** Webhook → GitLab `createProject` → Jira `createIssue` → Slack notify
- **PR Aging Report:** Cron → GitLab `listMergeRequests` → Function (filter old) → Slack report

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `owner and project are required for this operation`

**Cause:** `owner` or `project` fields are empty for a project-scoped operation.

**Solution:** Provide the exact GitLab namespace (username or group path) and the repository name. These are case-sensitive and must match the URL path: `https://gitlab.com/<owner>/<project>`.

#### `title is required for creating an issue`

**Cause:** `createIssue` or `createMergeRequest` called without a `title`.

**Solution:** Ensure the `title` field is non-empty. Check expression resolution if using `{{input.title}}`.

#### `GitLab API 401: Unauthorized`

**Cause:** The token is invalid, expired, or lacks the `api` scope.

**Solution:** Generate a new PAT at **GitLab → User Settings → Access Tokens**. Enable the `api` scope.

#### `GitLab API 404: Not Found`

**Cause:** The `owner/project` combination doesn't exist, or the `issue_iid`, `mr_id`, or `pipeline_id` is incorrect.

**Solution:** Verify the project path in the GitLab URL. Use `listIssues` or `listMergeRequests` to discover valid IDs.

#### `GitLab API 403: Forbidden`

**Cause:** The token belongs to a user who is not a member of the project, or the project is private.

**Solution:** Add the user as a project member with at least `Developer` role in GitLab project settings.

#### Branch name already exists (`400 Bad Request`)

**Cause:** `createBranch` was called for a branch that already exists.

**Solution:** Use `listBranches` to check existing branches before creating. Consider catching this error in a downstream function node.

### Token Scopes Reference

| Operations | Required Scope |
|------------|---------------|
| Issues, MRs, Branches | `api` |
| Pipelines / CI | `api` |
| Project creation | `api` |
| Read-only | `read_api` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [GitHub Action](./github.md) - Equivalent operations for GitHub repositories
- [Jenkins Action](./jenkins.md) - Trigger and monitor Jenkins CI builds
- [Jira Action](./jira.md) - Link GitLab operations to Jira issues and transitions

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->

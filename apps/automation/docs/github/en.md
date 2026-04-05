---
node_id: "github-action"
title: "GitHub Action"
description: "Manage GitHub issues, pull requests, branches, and GitHub Actions workflows via the Octokit REST API"
category: "integrations"
subcategory: "devops"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - github
  - devops
  - issues
  - pull-requests
  - ci-cd
  - branches
  - actions
  - action
related_nodes:
  - gitlab-action
  - jenkins-action
  - jira-action
  - discord-action
  - slack-action
---

<!-- SECTION: header -->
# GitHub Action

> **Category:** Integrations | **Type:** Action Node

Interact with GitHub repositories — manage issues, pull requests, and branches, trigger and monitor GitHub Actions workflows, and comment on issues, all via the official `@octokit/rest` SDK.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GitHub Action** node uses the `@octokit/rest` Octokit client authenticated with a GitHub Personal Access Token (PAT) or GitHub App token. It supports 17 operations across four areas: issues, workflows (GitHub Actions), branches, and pull requests.

All list operations support pagination via `per_page` and `page`. Responses are normalized and formatted for clean downstream consumption.

### Key Features

- **17 Operations:** Full coverage of issues, PRs, branches, and GitHub Actions
- **Octokit SDK:** Official GitHub REST client — typed, reliable, and rate-limit aware
- **Formatted Responses:** Raw Octokit data normalized to clean, minimal objects
- **Pagination:** `per_page` and `page` on all list operations
- **Expression Support:** All key fields (`issue_number`, `pull_number`, `branch`, `run_id`, etc.) accept dynamic expressions
- **Workflow Dispatch:** Trigger `workflow_dispatch` events to start GitHub Actions manually

### Use Cases

- Automatically create a GitHub issue when a monitoring alert fires
- Trigger a `workflow_dispatch` pipeline after a Jira issue is closed
- Close a PR automatically when its associated feature branch is merged
- Post a build summary as a comment on a GitHub issue
- List all open PRs targeting `main` as part of a release checklist workflow
- Create a feature branch from `develop` when a new Jira sprint starts

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | `string` | ✅ Yes | GitHub Personal Access Token (PAT) or GitHub App installation token |
| `owner` | `string` | ✅ Yes | Repository owner (GitHub username or organization name) |
| `repo` | `string` | ✅ Yes | Repository name |

> The token must have the relevant scopes: `repo` for most operations, `actions` for workflow operations.

### Operation Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to execute |
| `issue_number` | `number` | Conditional | Issue number — required for `getIssue`, `updateIssue`, `addComment` |
| `pull_number` | `number` | Conditional | PR number — required for `getPR`, `updatePR` |
| `title` | `string` | Conditional | Title — required for `createIssue`, `createPR`; optional for updates |
| `body` | `string` | Conditional | Body text — for `createIssue`, `updateIssue`, `createPR`, `updatePR` |
| `state` | `enum` | Conditional | State filter/update: `open`, `closed`, `all` |
| `comment` | `string` | Conditional | Comment text — required for `addComment` |
| `workflow_file` | `string` | Conditional | Workflow filename (e.g. `ci.yml`) — required for `triggerWorkflow`, `listWorkflowRuns` |
| `ref` | `string` | Conditional | Branch/tag ref — required for `triggerWorkflow` |
| `run_id` | `number` | Conditional | Workflow run ID — required for `getWorkflowRun`, `cancelWorkflowRun` |
| `branch` | `string` | Conditional | Branch name — required for `createBranch`, `deleteBranch`; optional filter for `listWorkflowRuns` |
| `source_branch` | `string` | Conditional | Branch to create from — required for `createBranch` |
| `pr_source_branch` | `string` | Conditional | PR head branch — required for `createPR`; optional filter for `listPRs` |
| `pr_target_branch` | `string` | Conditional | PR base branch — required for `createPR`; optional for `updatePR` / filter for `listPRs` |
| `status` | `enum` | Conditional | Workflow run status filter for `listWorkflowRuns` (see values below) |
| `per_page` | `number` | ❌ No | Results per page for list operations |
| `page` | `number` | ❌ No | Page number for list operations |
| `direction` | `enum` | ❌ No | Sort direction for `listIssues`: `asc` or `desc` |

### `status` Values (for `listWorkflowRuns`)

`completed` · `in_progress` · `queued` · `action_required` · `cancelled` · `failure` · `neutral` · `skipped` · `stale` · `success` · `timed_out` · `requested` · `waiting` · `pending`

### Available Operations

#### Issues

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listIssues` | — | List repository issues with optional `state` and `direction` filter |
| `getIssue` | `issue_number` | Get full details of a specific issue |
| `createIssue` | `title` | Create a new issue (optional: `body`) |
| `updateIssue` | `issue_number` | Update title, body, or state of an issue |
| `addComment` | `issue_number`, `comment` | Add a comment to an issue |

#### GitHub Actions Workflows

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listWorkflows` | — | List all workflows defined in the repository |
| `triggerWorkflow` | `workflow_file`, `ref` | Trigger a `workflow_dispatch` event on a workflow |
| `listWorkflowRuns` | `workflow_file` | List runs of a specific workflow (filterable by `status`, `branch`) |
| `getWorkflowRun` | `run_id` | Get details of a specific workflow run |
| `cancelWorkflowRun` | `run_id` | Cancel a running workflow |

#### Branches

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listBranches` | — | List all branches in the repository |
| `createBranch` | `branch`, `source_branch` | Create a new branch from an existing one |
| `deleteBranch` | `branch` | Delete a branch |

#### Pull Requests

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `listPRs` | — | List pull requests (filterable by `state`, `pr_source_branch`, `pr_target_branch`) |
| `getPR` | `pull_number` | Get details of a specific pull request |
| `createPR` | `title`, `pr_source_branch`, `pr_target_branch` | Create a new pull request |
| `updatePR` | `pull_number` | Update title, body, base branch, or state of a PR |

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
  "count": 3,
  "issues": [
    { "number": 42, "title": "Fix login bug", "state": "open", "created_at": "2026-03-01T...", "url": "https://github.com/...", "author": "driss" }
  ]
}
```

**`getIssue`**
```json
{
  "number": 42,
  "title": "Fix login bug",
  "state": "open",
  "body": "The login page crashes on Safari.",
  "created_at": "2026-03-01T...",
  "updated_at": "2026-03-10T...",
  "url": "https://github.com/...",
  "author": "driss",
  "comments_count": 3,
  "labels": ["bug", "frontend"]
}
```

**`createIssue` / `createPR`**
```json
{
  "success": true,
  "number": 43,
  "title": "New feature request",
  "url": "https://github.com/...",
  "created_at": "2026-03-11T..."
}
```

**`addComment`**
```json
{
  "success": true,
  "comment_id": 987654321,
  "comment_url": "https://github.com/.../issues/42#issuecomment-987654321",
  "created_at": "2026-03-11T...",
  "author": "fusion-bot"
}
```

**`triggerWorkflow`**
```json
{ "status": "workflow_dispatched" }
```

**`getWorkflowRun`**
```json
{
  "id": 12345678,
  "name": "CI Pipeline",
  "status": "completed",
  "conclusion": "success",
  "created_at": "2026-03-11T...",
  "updated_at": "2026-03-11T...",
  "url": "https://github.com/.../actions/runs/12345678",
  "branch": "main",
  "commit": "a3f8c2d"
}
```

**`createBranch`**
```json
{
  "success": true,
  "branch": "feature/new-feature",
  "sha": "a3f8c2d",
  "url": "https://api.github.com/repos/.../git/refs/heads/feature/new-feature"
}
```

**`getPR`**
```json
{
  "number": 15,
  "title": "Add new auth module",
  "state": "open",
  "body": "This PR adds...",
  "url": "https://github.com/...",
  "source_branch": "feature/auth",
  "target_branch": "main",
  "mergeable": true,
  "merged": false,
  "created_at": "2026-03-10T...",
  "author": "driss"
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
  "token": "ghp_your_token",
  "owner": "my-org",
  "repo": "my-repo",
  "operation": "createIssue",
  "title": "🚨 {{input.alertName}} triggered on {{input.server}}",
  "body": "**Alert:** {{input.alertName}}\n**Server:** {{input.server}}\n**Time:** {{input.triggeredAt}}\n**Details:** {{input.details}}"
}
```

---

### Example: Trigger a Deployment Workflow

**Configuration:**
```json
{
  "token": "ghp_your_token",
  "owner": "my-org",
  "repo": "my-repo",
  "operation": "triggerWorkflow",
  "workflow_file": "deploy.yml",
  "ref": "main"
}
```

**Output:**
```json
{ "status": "workflow_dispatched" }
```

---

### Example: Create a Feature Branch

**Configuration:**
```json
{
  "token": "ghp_your_token",
  "owner": "my-org",
  "repo": "my-repo",
  "operation": "createBranch",
  "branch": "feature/{{input.featureName}}",
  "source_branch": "develop"
}
```

---

### Example: List Open PRs Targeting Main

**Configuration:**
```json
{
  "token": "ghp_your_token",
  "owner": "my-org",
  "repo": "my-repo",
  "operation": "listPRs",
  "state": "open",
  "pr_target_branch": "main",
  "per_page": 20
}
```

---

### Example: Comment on Issue After Processing

**Configuration:**
```json
{
  "token": "ghp_your_token",
  "owner": "my-org",
  "repo": "my-repo",
  "operation": "addComment",
  "issue_number": "{{input.issue_number}}",
  "comment": "✅ Processed automatically by Fusion. Reference: **{{input.jobId}}**"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Jira → GitHub → Discord

When a Jira issue is moved to "In Review", create a GitHub branch, open a PR, and notify Discord.

```json
{
  "nodes": [
    {
      "id": "jira-webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "check-transition",
      "type": "function",
      "config": {
        "code": "if (input.transition.to.name !== 'In Review') return null; return { issueKey: input.issue.key, summary: input.issue.fields.summary };"
      }
    },
    {
      "id": "create-branch",
      "type": "github-action",
      "config": {
        "token": "ghp_your_token",
        "owner": "my-org",
        "repo": "my-repo",
        "operation": "createBranch",
        "branch": "review/{{input.issueKey}}",
        "source_branch": "develop"
      }
    },
    {
      "id": "create-pr",
      "type": "github-action",
      "config": {
        "token": "ghp_your_token",
        "owner": "my-org",
        "repo": "my-repo",
        "operation": "createPR",
        "title": "{{input.issueKey}}: {{input.summary}}",
        "pr_source_branch": "review/{{input.issueKey}}",
        "pr_target_branch": "main",
        "body": "Linked Jira issue: {{input.issueKey}}"
      }
    },
    {
      "id": "notify",
      "type": "discord-action",
      "config": {
        "operation": "sendEmbed",
        "channelId": "1234567890123456789",
        "embed": {
          "title": "🔀 PR created for {{input.issueKey}}",
          "description": "{{input.summary}}\n[View PR]({{input.url}})",
          "color": "blue"
        }
      }
    }
  ]
}
```

### Common Patterns

- **Alert → Issue:** Monitoring webhook → GitHub `createIssue` → Slack notify
- **CI Gate:** GitHub `getWorkflowRun` (poll) → Function (check conclusion) → Jira `transitionIssue`
- **Sprint Start:** Jira sprint event → GitHub `createBranch` (from `develop`)
- **PR Cleanup:** Cron → GitHub `listPRs` (stale open) → GitHub `updatePR` (close)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `owner is required` / `repo is required`

**Cause:** `owner` or `repo` field is empty.

**Solution:** Provide the exact GitHub username or org name and the exact repository name (no spaces, case-sensitive).

#### `title is required` (createIssue / createPR)

**Cause:** The `title` field was left empty or not resolved from an expression.

**Solution:** Ensure the `title` field is non-empty. Check expression resolution if using `{{input.title}}`.

#### `HttpError: 422 Unprocessable Entity` (createBranch)

**Cause:** The branch name already exists or contains invalid characters.

**Solution:** Check that the branch doesn't already exist with `listBranches`. Use URL-safe characters in branch names.

#### `HttpError: 404 Not Found` (workflow operations)

**Cause:** `workflow_file` doesn't match any `.github/workflows/*.yml` file in the repo, or `run_id` doesn't exist.

**Solution:** Use `listWorkflows` to confirm the exact filename. Verify `run_id` with `listWorkflowRuns`.

#### Rate limiting (`HttpError: 403`)

**Cause:** GitHub API rate limit reached (60 req/hr for unauthenticated, 5000 req/hr for PAT).

**Solution:** Use a PAT with appropriate scopes. Add delays between bulk operations using Function nodes.

### Token Scopes Reference

| Operations | Required Token Scope |
|------------|---------------------|
| Issues, PRs, Branches | `repo` |
| GitHub Actions | `repo` + `actions:write` |
| Read-only (list, get) | `repo:read` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [GitLab Action](./gitlab.md) - Equivalent operations for GitLab repositories
- [Jenkins Action](./jenkins.md) - Trigger and monitor Jenkins CI builds
- [Jira Action](./jira.md) - Link GitHub operations to Jira issues and transitions

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->

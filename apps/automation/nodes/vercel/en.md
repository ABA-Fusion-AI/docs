---
node_id: "vercel"
title: "Vercel"
description: "Manage Vercel deployments: list projects, trigger a deployment from a Git repository, and retrieve the status of an existing deployment."
category: "Cloud / Deployment"
subcategory: "Cloud & Hosting"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - vercel
  - deployment
  - hosting
  - devops
  - ci-cd
  - git
related_nodes:
  - firebase-hosting
  - netlify
  - heroku
  - http-request
---

<!-- SECTION: overview -->
# Vercel

> **Category:** Cloud / Deployment &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage your **Vercel** projects and deployments directly from a Fusion workflow using the Vercel REST API (`https://api.vercel.com`). The node supports three operations:
- **`list_projects`:** Returns all projects available to the access token.
- **`create_deployment`:** Triggers a new deployment of a project from a GitHub repository and branch/commit.
- **`get_deployment`:** Retrieves the details and current state of a deployment by its ID.

### Use Cases

- **Automated deploys:** Trigger a production or preview deployment after a successful test run or a merged pull request.
- **Deployment monitoring:** Poll a deployment's state and notify a team channel when it is `READY` or `ERROR`.
- **Project inventory:** List all Vercel projects of an account or team for reporting or auditing.
- **Scheduled redeploys:** Rebuild a site on a schedule to refresh statically generated content.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `token` | `string` | Yes | — | Vercel Access Token. Supports expressions. |
| `operation` | `enum` | Yes | `list_projects` | Operation to perform: `list_projects`, `create_deployment` or `get_deployment`. |
| `projectId` | `string` | For `create_deployment` | — | Vercel project ID (e.g. `prj_...`). |
| `projectName` | `string` | For `create_deployment` | — | Vercel project name. Used as the deployment `name`. |
| `gitRepo` | `string` | For `create_deployment` | — | GitHub repository to deploy, in `owner/repo` format. |
| `repoId` | `string` | For `create_deployment` | — | Numeric GitHub repository ID (e.g. `123456789`). |
| `gitRef` | `string` | No | `main` | Branch name or commit SHA to deploy. |
| `deploymentId` | `string` | For `get_deployment` | — | ID of the deployment to retrieve (e.g. `dpl_...`). |

### Operations & Requirements

| Operation | Vercel Endpoint | Required Fields |
|-----------|-----------------|-----------------|
| `list_projects` | `GET /v9/projects` | `token` |
| `create_deployment` | `POST /v13/deployments` | `token`, `projectId`, `projectName`, `gitRepo`, `repoId` (`gitRef` optional) |
| `get_deployment` | `GET /v13/deployments/{deploymentId}` | `token`, `deploymentId` |

### Authentication & Access Token

1. Log in to the **Vercel Dashboard** (`https://vercel.com`).
2. Open **Account Settings → Tokens**.
3. Click **Create Token**, choose its scope (personal account or team) and an expiration date.
4. Copy the token and paste it into the `token` field (or store it in Fusion Secrets).

> The GitHub repository used in `create_deployment` must already be connected to Vercel through the Vercel GitHub integration.

### Where to find the IDs

- **Project ID / Project Name:** Project → **Settings → General** (`Project ID` and `Project Name`). The name also appears in the URL `https://vercel.com/{team}/{project-name}`.
- **Repo ID:** Numeric ID of the GitHub repository. Open `https://api.github.com/repos/{owner}/{repo}` and copy the `id` field (for a private repository, use an authenticated call, e.g. `gh api repos/{owner}/{repo} --jq .id`).
- **Deployment ID:** Returned in the `id` field of a `create_deployment` response, or in the **Deployments** tab of the project.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Upstream values can be used through expressions (e.g. `{{ input.ref }}`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Raw JSON response returned by the Vercel API. |
| `error` | `Error` | Emitted when a required field is missing or on runtime errors. |

> The node returns the Vercel API response as-is. An API-level error (invalid token, unknown project…) is returned on `success` as an object containing an `error` field.

### Output Examples

#### `list_projects`

```json
{
  "projects": [
    {
      "id": "prj_12HKQaOmR5t5Uy6vdcQsNIiZgHGB",
      "name": "my-website",
      "framework": "nextjs",
      "createdAt": 1731840000000,
      "updatedAt": 1731926400000
    }
  ],
  "pagination": {
    "count": 1,
    "next": null,
    "prev": null
  }
}
```

#### `create_deployment`

```json
{
  "id": "dpl_89qyp1cskzkLrVicDaZoDbjyHuDJ",
  "name": "my-website",
  "url": "my-website-abc123.vercel.app",
  "readyState": "QUEUED",
  "target": null,
  "createdAt": 1731926400000
}
```

#### `get_deployment`

```json
{
  "id": "dpl_89qyp1cskzkLrVicDaZoDbjyHuDJ",
  "name": "my-website",
  "url": "my-website-abc123.vercel.app",
  "readyState": "READY",
  "createdAt": 1731926400000,
  "ready": 1731926465000
}
```

#### API Error (returned on `success`)

```json
{
  "error": {
    "code": "forbidden",
    "message": "Not authorized"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: List Vercel projects
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Vercel Node:** Calls the Vercel API with the selected operation (`list_projects`).
3. **Log Node:** Displays the list of projects returned by Vercel.

### Example Configuration

```json
{
  "token": "{{ secrets.VERCEL_TOKEN }}",
  "operation": "create_deployment",
  "projectId": "prj_12HKQaOmR5t5Uy6vdcQsNIiZgHGB",
  "projectName": "my-website",
  "gitRepo": "my-org/my-website",
  "repoId": "123456789",
  "gitRef": "main"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your Vercel `token` in Fusion's **Secrets** vault. Do not commit hardcoded tokens into version control.

- Create tokens with the narrowest scope possible (a single team rather than the whole account).
- Set an expiration date on tokens and rotate them regularly.
- `create_deployment` can publish code to production: restrict who can run workflows that use it.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Project ID, Project Name, Git Repo, and Repo ID are required.`
- **Cause:** `operation` is `create_deployment` but one of `projectId`, `projectName`, `gitRepo` or `repoId` is empty.
- **Solution:** Fill in all four fields (`gitRepo` in `owner/repo` format, `repoId` as the numeric GitHub ID).

#### `Deployment ID required.`
- **Cause:** `operation` is `get_deployment` but `deploymentId` is empty.
- **Solution:** Provide the deployment ID, for example from a previous `create_deployment` output (`{{ input.id }}`).

#### `error.code: "forbidden"` / `"invalidToken"`
- **Cause:** The token is invalid, expired, or has no access to the project's team.
- **Solution:** Generate a new token with the correct scope in **Account Settings → Tokens**.

#### `error.code: "incorrect_git_source_info"` / repository not found
- **Cause:** The repository is not connected to Vercel, or `gitRepo` / `gitRef` is wrong.
- **Solution:** Connect the repository through the Vercel GitHub integration and check `gitRepo`, `repoId` and `gitRef`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: operations, parameter reference, output schemas, and workflow integration |

<!-- /SECTION: changelog -->

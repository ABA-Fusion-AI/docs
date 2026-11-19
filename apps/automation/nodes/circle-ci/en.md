---
node_id: "circle-ci"
title: "CircleCI"
description: "Manage CircleCI projects, runs, workflows, jobs and environment variables with the CircleCI API v3 (and v2 for env vars and checkout keys)."
category: "Cloud / Deployment"
subcategory: "CI/CD"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - circleci
  - ci-cd
  - devops
  - builds
  - pipelines
related_nodes:
  - vercel
  - github
  - firebase-hosting
  - http-request
---

<!-- SECTION: overview -->
# CircleCI

> **Category:** Cloud / Deployment &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Control **CircleCI** from a Fusion workflow: read projects and pipelines, trigger runs, follow workflows and jobs, cancel or rerun workflows, and manage project environment variables. The node uses the **CircleCI API v3** (`https://circleci.com/api/v3`), and the **API v2** for environment variables and checkout keys.

### Use Cases

- **Deploy on demand:** Trigger a CircleCI run when a form is approved, a release is tagged or a schedule fires.
- **Build monitoring:** Get the workflows and jobs of a run and notify a channel when a job fails.
- **Self-healing pipelines:** Rerun a failed workflow automatically (`is_from_failed`).
- **Secrets rotation:** Create or delete project environment variables from a workflow.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Common Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiToken` | `string` | Yes | — | CircleCI personal API token. |
| `baseUrl` | `string` | No | `https://circleci.com/api/v3` | API v3 base URL (change only for CircleCI Server). |
| `operation` | `enum` | Yes | `getMe` | Operation to perform (see below). |

### Operations

| Operation | What it does | Required | Optional |
|-----------|--------------|----------|----------|
| `getMe` | Current user | — | — |
| `getProjects` | Projects of an org, or followed projects | — | `orgId` |
| `getProject` | One project | `projectId` | — |
| `getProjectSettings` | Advanced settings of a project | `projectId` | — |
| `getProjectPipelines` | Pipeline definitions of a project | `projectId` | — |
| `getProjectBuilds` | Runs of a project | `projectId` | — |
| `triggerBuild` | Starts a new run | `projectId`, `pipelineId` | `branch`, `buildData` (`parameters`, `checkout`, `config`) |
| `getBuild` | Workflows of a run (or the pipeline if no run ID) | `runId` or `pipelineId` | — |
| `cancelBuild` | Cancels a workflow | `workflowId` | — |
| `retryBuild` | Reruns a workflow | `workflowId` | `rerunData` (`is_from_failed`, `is_ssh_enabled`, `jobs`) |
| `getJobDetails` | Jobs of a workflow | `workflowId` | — |
| `getBuildArtifacts` | Artifacts of a job | `jobId` | — |
| `getProjectJobMetrics` | Details and timings of a job | `jobId` | — |
| `getProjectWorkflowMetrics` | Searches the runs of a project | `projectId` | `workflowMetricsParams.include` (`workflows`, `workflows.jobs`) |
| `getEnvironmentVariables` | Lists env vars (values masked) | `projectSlug` | — |
| `createEnvironmentVariable` | Creates or replaces an env var | `projectSlug`, `envVarName`, `envVarValue` | — |
| `deleteEnvironmentVariable` | Deletes an env var | `projectSlug`, `envVarName` | — |
| `getCheckoutKeys` | Lists checkout keys | `projectSlug` | — |

> **Not available:** `clearBuildCache`, `cancelJob`, `createCheckoutKey` and `deleteCheckoutKey` are listed in the operation menu but always return an error, because CircleCI API v3 does not offer them. To stop a job, cancel its workflow with `cancelBuild`.

### Run → Workflow → Job

CircleCI v3 organizes builds in three levels; each operation returns the ID needed by the next one:

1. `triggerBuild` → **run** `id`
2. `getBuild` (`runId`) → **workflow** `id`
3. `getJobDetails` (`workflowId`) → **job** `id`
4. `getProjectJobMetrics` / `getBuildArtifacts` (`jobId`)

### Authentication & API Token

1. In CircleCI, open **User Settings → Personal API Tokens**.
2. Click **Create New Token** and copy it (it starts with `CCIPAT_`).
3. Paste it into `apiToken` (or store it in Fusion Secrets).

### Where to find the IDs

| ID | How to get it |
|----|---------------|
| `orgId` | `getProjects` → `references.org.id`, or **Organization Settings → Overview** |
| `projectId` | `getProjects`, or **Project Settings → Overview** |
| `pipelineId` | `getProjectPipelines` (pipeline definition, e.g. `build-and-test`) |
| `projectSlug` | `circleci/<orgId>/<projectId>` for GitHub App / GitLab projects, `gh/<owner>/<repo>` for GitHub OAuth projects (shown in **Project Settings → Overview**) |

### Requirements for `triggerBuild`

The selected `branch` must contain a valid `.circleci/config.yml`. If it is missing or invalid, the run is created but not executed (`current_outcome: not_run`) or the API answers `400 … config file .circleci/config.yml could not be located`.

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
| `success` | `object` | JSON response of CircleCI. Empty responses and `deleteEnvironmentVariable` return `{ "success": true }`. |
| `error` | `Error` | Emitted on missing fields, unsupported operations or API errors. |

### Output Examples

#### `triggerBuild`

```json
{
  "data": {
    "id": "0c42d7dc-b333-41fd-8ae0-f6b563b69e3c",
    "attributes": { "created_at": "2026-11-17T23:21:12.517Z" },
    "references": {
      "pipeline": { "id": "07a65443-543c-405e-945b-08302be0995b" },
      "project": { "id": "d9f4230c-a4c7-40a2-ab58-cfd5a221726f" }
    }
  }
}
```

#### `getBuild` (workflows of a run)

```json
{
  "data": [
    {
      "id": "e77a5c88-3f10-42bb-8e01-6fa6283718d7",
      "attributes": {
        "name": "build-and-test",
        "phase": "started",
        "current_outcome": "failed",
        "created_at": "2026-11-17T23:21:12.791Z"
      },
      "references": { "run": { "id": "0c42d7dc-b333-41fd-8ae0-f6b563b69e3c" } }
    }
  ]
}
```

#### `getProjectJobMetrics`

```json
{
  "data": {
    "id": "6dff1ccf-1727-4a77-874a-51f765598702",
    "attributes": {
      "name": "test",
      "type": "build",
      "phase": "ended",
      "outcome": "succeeded",
      "queued_at": "2026-11-17T23:21:13.243Z",
      "started_at": "2026-11-17T23:21:15.197Z",
      "ended_at": "2026-11-17T23:22:19.700Z"
    }
  }
}
```

#### `createEnvironmentVariable`

```json
{ "name": "FUSION_TEST", "value": "xxxxlo", "created_at": "2026-11-17T23:05:42.105Z" }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Trigger a CircleCI run
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **CircleCI Node:** Runs the selected operation.
3. **Log Node:** Displays the CircleCI response.

### Example Configuration

```json
{
  "apiToken": "{{ secrets.CIRCLECI_TOKEN }}",
  "operation": "triggerBuild",
  "projectId": "d9f4230c-a4c7-40a2-ab58-cfd5a221726f",
  "pipelineId": "07a65443-543c-405e-945b-08302be0995b",
  "branch": "main"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the CircleCI `apiToken` in Fusion's **Secrets** vault. Do not commit it into version control.

- A personal token has all the rights of your CircleCI user: create it with a dedicated account if possible.
- Environment variable values are sent in clear to CircleCI; never log them in the workflow.
- `triggerBuild` and `retryBuild` consume CircleCI credits.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `CircleCI operation failed: Project ID is required` (or another ID)
- **Cause:** The selected operation needs an ID that is empty.
- **Solution:** See the *Operations* table and *Where to find the IDs*.

#### `... (400): error fetching config - config file .circleci/config.yml could not be located in <branch>`
- **Cause:** The branch has no CircleCI config.
- **Solution:** Push `.circleci/config.yml` to this branch, or set `branch` to the branch that has it.

#### `... (400): failed to fetch config reference: <branch>`
- **Cause:** The branch does not exist in the repository.
- **Solution:** Check the branch name.

#### Run created with `current_outcome: not_run` and `Unable to parse YAML`
- **Cause:** Invalid `config.yml` (for example `jobs:` or `workflows:` written twice).
- **Solution:** Fix the YAML; each top-level key must appear only once.

#### `... is not available in CircleCI API v3` / `not supported for this CircleCI project type`
- **Cause:** `clearBuildCache`, `cancelJob`, `createCheckoutKey` or `deleteCheckoutKey` was selected.
- **Solution:** Use another operation (`cancelBuild` to stop a job's workflow).

#### `... (404): Not Found` on `getBuildArtifacts`
- **Cause:** Unknown job ID, or the job stored no artifacts.
- **Solution:** Use the ID of a job that runs `store_artifacts`.

#### `... (401)` / `Unauthorized`
- **Cause:** Invalid or revoked token.
- **Solution:** Create a new personal API token.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: operations, run/workflow/job model, IDs, output examples, troubleshooting |

<!-- /SECTION: changelog -->

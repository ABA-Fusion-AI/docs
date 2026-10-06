---
node_id: "rundeck"
title: "Rundeck"
description: "Rundeck job scheduler - List and run jobs, manage executions and projects."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags:
  - rundeck
  - jobs
  - automation
  - devops
  - integration
  - peer-only
related_nodes:
  - http-request
  - function
  - cron
---

<!-- SECTION: overview -->
# Rundeck

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Control a [Rundeck](https://www.rundeck.com) server from a workflow: list the jobs of a project, run a job, follow its execution, and read project information. The node calls the Rundeck REST API (`/api/41`) with an API token and works with Rundeck Community (open source) and commercial editions.

### Use Cases

- **Run operational jobs from a workflow:** Trigger a deployment, backup, or maintenance job when an event occurs.
- **Follow a job run:** Start a job and check its execution status (`running`, `succeeded`, `failed`).
- **Report on activity:** List the recent executions of a project and send a summary.
- **Discover jobs:** List the jobs of a project to get their IDs.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `listJobs` | `listJobs`, `runJob`, `getExecution`, `listExecutions`, or `getProject`. |
| `host` | `string` | Yes | — | Rundeck host **without** protocol, e.g. `rundeck.example.com` or `rundeck.example.com:4443`. The node always uses `https://`. Supports expressions. |
| `apiToken` | `string` | Yes | — | Rundeck user API token, sent as `X-Rundeck-Auth-Token`. Supports expressions. |
| `project` | `string` | No | — | Project name. Used by `listJobs`, `listExecutions`, `getProject`. Supports expressions. |
| `jobId` | `string` | No | — | Job UUID. Used by `runJob`. Supports expressions. |
| `executionId` | `string` | No | — | Execution ID (number). Used by `getExecution`. Supports expressions. |
| `jobArgs` | `string` (JSON) | No | — | Job options as a JSON object, e.g. `{"name":"Alice"}`. Used by `runJob`. Supports expressions. |
| `maxResults` | `number` | No | — | Maximum number of executions returned by `listExecutions`. |

### Available Operations

| Operation | Description | Rundeck request | Required |
|-----------|-------------|-----------------|----------|
| `listJobs` | List the jobs of a project. | `GET /api/41/project/<project>/jobs` | `project` |
| `runJob` | Run a job now, optionally with options. | `POST /api/41/job/<jobId>/run` | `jobId` |
| `getExecution` | Get the status and details of an execution. | `GET /api/41/execution/<executionId>` | `executionId` |
| `listExecutions` | List the executions of a project (most recent first). | `GET /api/41/project/<project>/executions?max=<maxResults>` | `project` |
| `getProject` | Get a project's details and configuration. | `GET /api/41/project/<project>` | `project` |

> The node does not check required fields before calling Rundeck. A missing value is sent as `undefined` and Rundeck answers `404` (see [Troubleshooting](#troubleshooting)).

### Getting an API Token

1. Sign in to Rundeck.
2. Click the **user icon** (top right) → **Profile**.
3. Under **User API Tokens**, click **+**, choose the roles and an expiration, and click **Generate New Token**.
4. Copy the token (it is shown only once) into `apiToken`.

The token has the permissions of the roles you select. Use a role that can read the project and run the required jobs.

### Finding IDs

- **Project name:** shown in the project list and in the URL (`/project/<name>/...`).
- **Job ID:** the UUID in the job page URL (`/job/show/<uuid>`), or the `id` returned by `listJobs`.
- **Execution ID:** the `id` returned by `runJob` or `listExecutions`, or the number in the execution URL.

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
| `success` | `object` \| `array` | The Rundeck API response for the operation. |
| `error` | `Error` | Rundeck API errors (`Rundeck API Error: <status> <statusText> <body>`) and network errors. |

### Output Examples

#### `listJobs`

```json
[
  {
    "id": "6cca6a90-1bee-4527-9a2d-45ec1fcd8f75",
    "name": "hello",
    "group": null,
    "project": "fusion",
    "description": "echo hello",
    "enabled": true,
    "scheduled": false,
    "permalink": "https://rundeck.example.com/project/fusion/job/show/6cca6a90-1bee-4527-9a2d-45ec1fcd8f75"
  }
]
```

#### `runJob`

```json
{
  "id": 1,
  "status": "running",
  "project": "fusion",
  "user": "admin",
  "date-started": { "unixtime": 1791281861955, "date": "2026-10-06T10:17:41Z" },
  "job": { "id": "6cca6a90-1bee-4527-9a2d-45ec1fcd8f75", "name": "hello", "group": "" },
  "permalink": "https://rundeck.example.com/project/fusion/execution/show/1"
}
```

Use the returned `id` with `getExecution` to follow the run.

#### `getExecution`

```json
{
  "id": 1,
  "status": "succeeded",
  "project": "fusion",
  "job": { "id": "6cca6a90-1bee-4527-9a2d-45ec1fcd8f75", "name": "hello" },
  "date-started": { "date": "2026-10-06T10:17:41Z" }
}
```

`status` is one of `running`, `succeeded`, `failed`, `aborted`, `timedout`, `failed-with-retry`, `scheduled`, or `other`.

#### `listExecutions`

```json
{
  "paging": { "count": 1, "total": 1, "offset": 0, "max": 5 },
  "executions": [
    { "id": 1, "status": "succeeded", "project": "fusion", "job": { "name": "hello" } }
  ]
}
```

#### `getProject`

```json
{
  "name": "fusion",
  "description": "",
  "created": "2026-10-06T10:06:59Z",
  "config": { "project.name": "fusion", "...": "..." }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Rundeck in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Rundeck Node:** Runs the configured operation (e.g. `listJobs`) on the Rundeck server.
3. **Log Node:** Displays the Rundeck response.

### Example Configurations

#### Run a job with options

```json
{
  "operation": "runJob",
  "host": "rundeck.example.com",
  "apiToken": "{{ secrets.RUNDECK_TOKEN }}",
  "jobId": "6cca6a90-1bee-4527-9a2d-45ec1fcd8f75",
  "jobArgs": "{\"name\":\"Alice\"}"
}
```

#### Check an execution

```json
{
  "operation": "getExecution",
  "host": "rundeck.example.com",
  "apiToken": "{{ secrets.RUNDECK_TOKEN }}",
  "executionId": "1"
}
```

#### List the last 5 executions

```json
{
  "operation": "listExecutions",
  "host": "rundeck.example.com",
  "apiToken": "{{ secrets.RUNDECK_TOKEN }}",
  "project": "fusion",
  "maxResults": 5
}
```

### Common Patterns

- **Event → Rundeck (`runJob`):** Run a remediation or deployment job when an alert or webhook arrives.
- **Rundeck (`runJob`) → Rundeck (`getExecution`):** Start a job, then check its execution with the returned `id` and branch on `status`.
- **Cron → Rundeck (`listExecutions`) → Function → Notification:** Send a daily report of failed executions.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `apiToken` in Fusion's **Secrets** system. Never paste tokens directly into workflow parameters or commit them to version control.

- Create the token with the least-privileged roles that can run the required jobs.
- Set an expiration on the token and rotate it regularly.
- `runJob` executes real commands on your Rundeck nodes — restrict which jobs the token can run with Rundeck ACL policies.
- The node only connects over HTTPS; expose Rundeck with a valid TLS certificate.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Rundeck API Error: 403 Forbidden` / `401 Unauthorized`
- **Cause:** The token is wrong, expired, or its roles are not allowed to access the project or run the job.
- **Solution:** Generate a new token with the right roles, or update the ACL policy.

#### `404 … Project does not exist: undefined` / `Execution does not exist: undefined`
- **Cause:** `project` (for `listJobs`, `listExecutions`, `getProject`), `jobId` (for `runJob`), or `executionId` (for `getExecution`) is empty.
- **Solution:** Fill in the field required by the operation (see [Available Operations](#available-operations)).

#### `404 … Project does not exist: <name>` / job not found
- **Cause:** Wrong project name or job UUID.
- **Solution:** Use `listJobs` to get the exact job IDs; project names are case-sensitive.

#### `fetch failed` / connection error
- **Cause:** `host` contains `https://` or `http://`, the server only serves plain HTTP, or it is not reachable from the Fusion environment.
- **Solution:** Put only the host name (and port) in `host`, and expose Rundeck over HTTPS.

#### Job runs without its options
- **Cause:** `jobArgs` is not valid JSON — the node ignores it silently and runs the job without options.
- **Solution:** Use a valid JSON object, e.g. `{"name":"Alice"}`.

#### `maxResults` has no effect
- **Cause:** `maxResults` is only used by `listExecutions`.
- **Solution:** Filter or slice `listJobs` results in a Function node.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Full documentation: parameters, operations, output examples, token setup, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->
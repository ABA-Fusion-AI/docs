---
node_id: "jenkins-action"
title: "Jenkins Action"
description: "Interact with Jenkins CI/CD — trigger builds, retrieve job and build information, manage job state, and inspect the build queue"
category: "integrations"
subcategory: "devops"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - jenkins
  - ci-cd
  - devops
  - builds
  - automation
  - action
related_nodes:
  - slack-action
  - jira-action
  - function
  - cron
---

<!-- SECTION: header -->
# Jenkins Action

> **Category:** Integrations | **Type:** Action Node

Connect to a Jenkins server and perform CI/CD operations — list and inspect jobs, trigger parameterized builds, retrieve build logs, stop running builds, manage job state, and monitor the build queue.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Jenkins Action** node authenticates against the Jenkins REST API using Basic Auth (username + API token) and executes one of ten available operations. It supports both simple and parameterized job builds, build console log retrieval, and full job lifecycle management (enable, disable, delete).

### Key Features

- **10 Operations:** getJob, listJobs, triggerBuild, getBuildInfo, getBuildConsole, stopBuild, deleteJob, enableJob, disableJob, getSystemInfo, getQueue
- **Parameterized Builds:** Pass key/value parameters to any Jenkins job
- **Console Logs:** Retrieve the full console output of any build
- **Build Targeting:** Reference builds by number or use `"last"` to target the latest build
- **System Inspection:** Retrieve Jenkins version, plugins, and executor info
- **Queue Monitoring:** Inspect the current build queue
- **Expression Support:** `jobName` and `buildNumber` accept dynamic expressions from upstream nodes

### Use Cases

- Trigger a Jenkins deployment job from a workflow after a Jira ticket status changes
- Check the status of the latest build before promoting an artifact
- Retrieve build console logs for post-processing or error detection
- Stop a runaway build and notify the team via Slack
- Enable or disable jobs as part of maintenance windows
- Monitor the Jenkins queue depth as a health check

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `baseURL` | `string` | ✅ Yes | Jenkins base URL (e.g. `https://jenkins.example.com`) |
| `username` | `string` | ✅ Yes | Jenkins username |
| `apiToken` | `string` | ✅ Yes | Jenkins API token — generate in **User → Configure → API Token** |

### Operation Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | — | Operation to execute (see below) |
| `jobName` | `string` | Conditional | — | Jenkins job name — required for all operations except `listJobs`, `getSystemInfo`, `getQueue` |
| `buildNumber` | `number` | Conditional | `"last"` | Build number — required for `stopBuild` (must be numeric), optional for `getBuildInfo` and `getBuildConsole` |
| `parameters` | `object` | ❌ No | — | Key/value pairs passed as build parameters for `triggerBuild` (e.g. `{ "BRANCH": "main", "ENV": "staging" }`) |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `getJob` | Retrieve job details including last build status and URL |
| `listJobs` | List all top-level jobs on the Jenkins server |
| `triggerBuild` | Trigger a build — with or without parameters |
| `getBuildInfo` | Get build metadata (status, duration, result) |
| `getBuildConsole` | Retrieve the full console log of a build |
| `stopBuild` | Abort a running build |
| `deleteJob` | Delete a job permanently |
| `enableJob` | Re-enable a disabled job |
| `disableJob` | Disable a job to prevent future builds |
| `getSystemInfo` | Retrieve Jenkins version, node info, and plugin list |
| `getQueue` | List all items currently waiting in the build queue |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — `jobName`, `buildNumber`, and `parameters` can also be resolved from here |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the Jenkins API call |
| `error` | `Error` | Emitted if the operation or connection fails |

### Output Schemas by Operation

**`getJob`**
```json
{
  "name": "deploy-production",
  "url": "https://jenkins.example.com/job/deploy-production/",
  "color": "blue",
  "buildable": true,
  "lastBuild": {
    "number": 42,
    "url": "https://jenkins.example.com/job/deploy-production/42/",
    "result": "SUCCESS"
  },
  "lastSuccessfulBuild": { "number": 42 },
  "lastFailedBuild": { "number": 38 }
}
```

**`listJobs`**
```json
[
  { "name": "deploy-production", "color": "blue", "url": "..." },
  { "name": "run-tests", "color": "red", "url": "..." },
  { "name": "build-docker", "color": "notbuilt", "url": "..." }
]
```

**`triggerBuild`**
```json
{
  "success": true,
  "jobName": "deploy-production",
  "queueLocation": "https://jenkins.example.com/queue/item/123/"
}
```

**`getBuildInfo`**
```json
{
  "number": 42,
  "result": "SUCCESS",
  "duration": 94200,
  "timestamp": 1741600000000,
  "url": "https://jenkins.example.com/job/deploy-production/42/",
  "building": false,
  "displayName": "#42",
  "description": null,
  "actions": []
}
```

**`getBuildConsole`**
```json
{
  "jobName": "deploy-production",
  "buildNumber": 42,
  "console": "Started by user admin\n[Pipeline] Start of Pipeline\n...\nFinished: SUCCESS"
}
```

**`stopBuild`**
```json
{
  "success": true,
  "jobName": "deploy-production",
  "buildNumber": 43
}
```

**`enableJob` / `disableJob` / `deleteJob`**
```json
{
  "success": true,
  "jobName": "deploy-production"
}
```

**`getSystemInfo`**
```json
{
  "version": "2.440.1",
  "url": "https://jenkins.example.com/",
  "nodeDescription": "the master Jenkins node",
  "numExecutors": 4,
  "mode": "NORMAL"
}
```

**`getQueue`**
```json
{
  "items": [
    {
      "id": 123,
      "task": { "name": "run-tests", "url": "..." },
      "why": "Waiting for next available executor",
      "inQueueSince": 1741600100000
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Trigger a Simple Build

Trigger the `run-tests` job with no parameters.

**Configuration:**
```json
{
  "baseURL": "https://jenkins.example.com",
  "username": "admin",
  "apiToken": "your-api-token",
  "operation": "triggerBuild",
  "jobName": "run-tests"
}
```

**Output:**
```json
{
  "success": true,
  "jobName": "run-tests",
  "queueLocation": "https://jenkins.example.com/queue/item/124/"
}
```

---

### Example: Trigger a Parameterized Build

Build a specific branch in a specific environment.

**Configuration:**
```json
{
  "baseURL": "https://jenkins.example.com",
  "username": "admin",
  "apiToken": "your-api-token",
  "operation": "triggerBuild",
  "jobName": "deploy-service",
  "parameters": {
    "BRANCH": "{{input.branch}}",
    "ENV": "staging",
    "VERSION": "{{input.version}}"
  }
}
```

---

### Example: Get the Latest Build Status

Check whether the last build of a job passed or failed.

**Configuration:**
```json
{
  "baseURL": "https://jenkins.example.com",
  "username": "admin",
  "apiToken": "your-api-token",
  "operation": "getBuildInfo",
  "jobName": "deploy-production"
}
```

> When `buildNumber` is omitted, the node defaults to `"last"` — the most recent build.

---

### Example: Retrieve Console Log

Get the console output of build `#42`.

**Configuration:**
```json
{
  "baseURL": "https://jenkins.example.com",
  "username": "admin",
  "apiToken": "your-api-token",
  "operation": "getBuildConsole",
  "jobName": "deploy-production",
  "buildNumber": 42
}
```

---

### Example: Stop a Running Build

Abort build `#43` of a job.

**Configuration:**
```json
{
  "baseURL": "https://jenkins.example.com",
  "username": "admin",
  "apiToken": "your-api-token",
  "operation": "stopBuild",
  "jobName": "deploy-production",
  "buildNumber": 43
}
```

---

### Example: Disable a Job for Maintenance

**Configuration:**
```json
{
  "baseURL": "https://jenkins.example.com",
  "username": "admin",
  "apiToken": "your-api-token",
  "operation": "disableJob",
  "jobName": "nightly-regression"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Jira Ticket → Jenkins Build → Slack Notification

When a Jira issue transitions to "Ready for Deploy", trigger a Jenkins build and post the result to Slack.

```json
{
  "nodes": [
    {
      "id": "jira-webhook",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "check-transition",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "if (input.transition?.name !== 'Ready for Deploy') return null; return input;"
      }
    },
    {
      "id": "trigger-jenkins",
      "type": "jenkins-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "baseURL": "https://jenkins.example.com",
        "username": "admin",
        "apiToken": "your-api-token",
        "operation": "triggerBuild",
        "jobName": "deploy-production",
        "parameters": { "ISSUE_KEY": "{{input.issue.key}}" }
      }
    },
    {
      "id": "notify-slack",
      "type": "slack-action",
      "position": { "x": 700, "y": 100 },
      "config": {
        "token": "xoxb-your-bot-token",
        "operation": "message",
        "messageOperation": "send",
        "messageChannelId": "C08XXXXXXXX",
        "text": "🚀 Build triggered for *{{input.issue.key}}* — <{{input.queueLocation}}|View queue>"
      }
    }
  ]
}
```

### Common Patterns

- **CI/CD Pipeline:** Webhook → Jenkins `triggerBuild` → Poll `getBuildInfo` → Notify [Slack](./slack-action.md)
- **Build Gate:** Jenkins `getBuildInfo` → Function (check `result === "SUCCESS"`) → Proceed or abort
- **Maintenance Window:** Cron (start of maintenance) → Jenkins `disableJob` → Cron (end) → Jenkins `enableJob`
- **Log Analysis:** Jenkins `getBuildConsole` → Function (grep for errors) → Jira `createIssue` on failure
- **Queue Monitor:** Cron → Jenkins `getQueue` → Alert if queue depth exceeds threshold

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to initialize Jenkins client`

**Cause:** `baseURL` is unreachable, or credentials are invalid.

**Solution:** Verify the URL (no trailing slash required but it is tolerated), `username`, and `apiToken`. Test the connection by opening `https://jenkins.example.com/api/json` in a browser while logged in.

#### `jobName is required`

**Cause:** The `jobName` field was left empty.

**Solution:** Provide the exact job name as it appears in the Jenkins dashboard. Job names are case-sensitive and must not include the `/job/` path prefix.

#### `buildNumber (numeric) is required for stopBuild`

**Cause:** `stopBuild` was called without a numeric `buildNumber`.

**Solution:** Provide the exact build number as a number (not a string). Use `getBuildInfo` to retrieve the current build number first.

#### `triggerBuild` succeeds but build never starts

**Cause:** The build was queued but no executors are available, or the job has a build throttle.

**Solution:** Use `getQueue` to confirm the build is in the queue. Check Jenkins executor availability with `getSystemInfo`.

#### `404 Not Found` on job operations

**Cause:** The job does not exist at the top level — it may be inside a folder.

**Solution:** For jobs inside folders, use the full path notation: `FolderName/JobName`.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check `baseURL` and network access |
| `401` | Unauthorized | Verify `username` and `apiToken` |
| `403` | Forbidden | User lacks permission for the operation |
| `404` | Job or build not found | Check `jobName` (case-sensitive, folder paths) |
| `409` | Build already stopped | Build completed before the stop request |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Slack Action](./slack-action.md) - Post build notifications and results to Slack
- [Jira Action](./jira.md) - Create issues or transition tickets on build events
- [Cron](./cron.md) - Schedule periodic builds or maintenance operations

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->

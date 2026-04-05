---
node_id: "clickup"
title: "ClickUp"
description: "Create, read, update and delete ClickUp tasks, comments, lists, folders, checklists, goals, task-list links, and tags."
category: "actions"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - action
  - integration
  - clickup
  - productivity
  - project management
related_nodes: []
---

<!-- SECTION: header -->
# ClickUp

> **Category:** Actions | **Type:** Action Node

Connect to your ClickUp workspace to automate project management tasks. You can manage tasks, comments, lists, folders, checklists, goals, tags, and more directly from your workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **ClickUp** node allows you to interact with the ClickUp API. It acts as an integration action node, performing operations on various resources within ClickUp.

### Key Features

- Manage multiple ClickUp resources (Tasks, Lists, Folders, Comments, Checklists, Goals, Tags)
- Full CRUD operations supported for most resources
- Easy association of tasks, lists, tags, and goals

### Use Cases

- Automatically create a task in ClickUp when a new lead is added in the CRM.
- Sync tasks between ClickUp and other project management tools.
- Add tags to a task automatically based on specific keywords.
- Generate routine checklists when a new project folder is created.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `baseUrl` | `string` | No | `https://api.clickup.com/api/v2` | ClickUp API Base URL |
| `apiKey` | `string` | ✅ Yes | — | ClickUp Personal API Token |

### Resources and Operations

Select a resource and the desired operation.

| Resource | Available Operations |
|----------|----------------------|
| **Task** | `create`, `get`, `getAll`, `update`, `delete` |
| **Comment** | `create`, `getAll`, `update`, `delete` |
| **List** | `create`, `get`, `getAll`, `getMembers`, `update`, `delete` |
| **Folder** | `create`, `get`, `getAll`, `update`, `delete` |
| **Checklist** | `create`, `getAll`, `delete` |
| **Checklist Item** | `create`, `update`, `delete` |
| **Goal** | `create`, `get`, `getAll`, `update`, `delete` |
| **Goal Key Result** | `create`, `update`, `delete` |
| **Task List** | `add`, `remove` |
| **Tag** | `create`, `getAll`, `delete` |
| **Task Tag** | `add`, `delete` |

### Key Parameters by Resource

Depending on the chosen resource and operation, different parameters will become available. Commonly required identifiers include:
- `folderSpaceId`: Space ID (e.g., when creating a Folder)
- `folderId`: Folder ID
- `listId`: List ID (e.g., when creating a Task)
- `taskId`: Task ID (e.g., when updating a Task)
- `commentTaskId` or `commentListId`: Required when interacting with comments
- `goalTeamId`: Team/Workspace ID (e.g., when creating a Goal)

Refer to the node configuration panel for field-specific requirements based on the selected operation.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This node receives input data from previous nodes. Input data can be dynamically mapped to configuration fields, such as passing a dynamic task name.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Returns the JSON response from the ClickUp API based on the operation performed. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Create a Task

Create a new task in a specific list.

**Configuration:**
```json
{
  "resource": "task",
  "taskOperation": "create",
  "listId": "900900912",
  "taskName": "Review latest PRs",
  "taskDescription": "Review and merge pull requests for the backend repo."
}
```

### Advanced Example: Add a Comment to a Task

Add a comment and optionally notify all assignees.

**Configuration:**
```json
{
  "resource": "comment",
  "commentOperation": "create",
  "commentTaskId": "86bc3q",
  "commentText": "I have verified the changes on staging.",
  "commentNotifyAll": true
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Sync Form Submission to Task

```json
{
  "nodes": [
    {
      "id": "webhook-trigger",
      "type": "webhook",
      "position": { "x": 100, "y": 100 },
      "config": {
        "webhookId": "custom-id"
      }
    },
    {
      "id": "clickup-create-task",
      "type": "clickup",
      "position": { "x": 300, "y": 100 },
      "config": {
        "apiKey": "pk_...",
        "resource": "task",
        "taskOperation": "create",
        "listId": "123456",
        "taskName": "={{ $json.body.name }} Form Submission",
        "taskDescription": "={{ $json.body.message }}"
      }
    }
  ],
  "connections": [
    {
      "source": "webhook-trigger",
      "sourceOutput": "output",
      "target": "clickup-create-task",
      "targetInput": "input"
    }
  ]
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Missing Required IDs
**Cause:** ClickUp's API depends heavily on hierarchical IDs (Space ID, Folder ID, List ID, Task ID). If an operation requires a parent ID that is not provided, the request will fail.
**Solution:** Check the node configuration and ensure you have mapped or entered the correct `listId`, `taskId`, etc.

#### Invalid API Key
**Cause:** The provided Personal API Token is incorrect or expired.
**Solution:** Generate a new Personal API Token from your ClickUp account settings (Apps > API Token) and update your node's connection settings.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `401 Unauthorized` | Invalid Authentication | Verify your Personal API Token is correct |
| `404 Not Found` | Resource Not Found | Ensure the provided Space/Folder/List/Task ID exists and is accessible |
| `400 Bad Request`| Missing required parameters | Check if name or required identifiers are provided for the chosen operation |

<!-- /SECTION: troubleshooting -->

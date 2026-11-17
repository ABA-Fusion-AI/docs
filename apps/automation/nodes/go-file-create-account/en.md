---
node_id: "go-file-create-account"
title: "GoFile Create Account"
description: "Create a new GoFile.io guest account and get its token, root folder and storage limits."
category: "Storage & Files"
subcategory: "File Sharing"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - gofile
  - file-sharing
  - storage
  - upload
related_nodes:
  - go-file-upload-file
  - go-file-get-content
  - go-file-delete-content
  - go-file-get-best-server
---

<!-- SECTION: overview -->
# GoFile Create Account

> **Category:** Storage & Files &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Create a **GoFile.io guest account** and get its **token**. The token is then used by *Upload File*, *Get Content* and *Delete Content* to upload into the same account and manage the files.

No parameter is needed.

This node is part of the GoFile family: *Create Account*, *Get Best Server*, *Upload File*, *Get Content*, *Delete Content*.

### Use Cases

- **Isolated storage:** Create a dedicated account for a workflow or a client.
- **Test setups:** Get a throwaway token to test the other GoFile nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

This node has no parameters.

### Authentication

Creates the account: no token needed.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` / `string` | Triggers the node. Upstream values can be used through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Result of the operation (see below). |
| `error` | `Error` | Emitted on missing fields or GoFile errors. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | `string` | `ok` on success. |
| `token` | `string` | Token of the new account. **Save it**: it cannot be retrieved later. |
| `email` | `string` | Account email (empty for guests). |
| `tier` | `string` | Account tier (e.g. `guest`). |
| `root_folder` | `string` | ID of the account's root folder. |
| `files_count / files_count_limit` | `number` | Files stored / allowed. |
| `total_size / total_size_limit` | `number` | Storage used / allowed, in bytes. |
| `credit` | `number` | Account credit. |
| `note` | `string` | Reminder to save the token. |

### Output Example

```json
{
  "status": "ok",
  "token": "qWeRtY123456",
  "email": "",
  "tier": "guest",
  "root_folder": "1234abcd-5678-90ef-1234-567890abcdef",
  "files_count": 0,
  "files_count_limit": 0,
  "total_size": 0,
  "total_size_limit": 0,
  "credit": 0,
  "note": "Save this token to manage your uploads and folders"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: GoFile Create Account
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **GoFile Create Account Node:** Calls the GoFile API.
3. **Log Node:** Displays the result.

A typical chain is **Create Account → Upload File → Get Content → Delete Content**, passing the `token` and IDs with expressions.

### Example Configuration

```json
{}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store GoFile tokens in Fusion's **Secrets** vault. Do not commit hardcoded tokens into version control.

- A GoFile token gives full access to the account's files: treat it like a password.
- Files uploaded to GoFile are reachable by anyone who has the download link.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to create account: Failed to create account`
- **Cause:** GoFile refused to create the account (status other than `ok`).
- **Solution:** Retry later; GoFile may limit guest account creation.

#### `... Rate Limited: Too many requests` / `Server Error: GoFile.io is experiencing issues`
- **Cause:** GoFile rate limit or temporary outage. The node already retried up to 3 times.
- **Solution:** Wait and run the workflow again.

#### `... Request timeout after 60000ms`
- **Cause:** GoFile did not answer in time.
- **Solution:** Retry later.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation |

<!-- /SECTION: changelog -->

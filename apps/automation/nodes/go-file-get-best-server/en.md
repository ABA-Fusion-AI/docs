---
node_id: "go-file-get-best-server"
title: "GoFile Get Best Server"
description: "Get the best available GoFile.io storage server for uploads, optionally in a given zone."
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
  - go-file-create-account
  - go-file-upload-file
  - go-file-get-content
  - go-file-delete-content
---

<!-- SECTION: overview -->
# GoFile Get Best Server

> **Category:** Storage & Files &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Ask **GoFile.io** (`https://api.gofile.io/servers`) which storage server to use for uploads. The node returns the server name and its upload URL. If GoFile lists no server, it returns the **global upload endpoint** (`https://upload.gofile.io/uploadfile`), which picks a server automatically.

No token is required.

This node is part of the GoFile family: *Create Account*, *Get Best Server*, *Upload File*, *Get Content*, *Delete Content*.

### Use Cases

- **Upload routing:** Choose an upload server close to your users (Europe or North America).
- **Pre-check:** Make sure GoFile is reachable before a batch of uploads.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `zone` | `enum` | No | — | Preferred zone: `eu` (Europe) or `na` (North America). Empty = any zone. |

### Authentication

Not needed.

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
| `server` | `string \| null` | Server name (e.g. `store1`), or `null` if none was listed. |
| `upload_url` | `string` | Upload URL to use (server URL or the global endpoint). |
| `note` | `string` | Short usage note. |

### Output Example

```json
{
  "status": "ok",
  "server": "store1",
  "upload_url": "https://store1.gofile.io/contents/uploadfile",
  "note": "Use this server for your file uploads"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: GoFile Get Best Server
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **GoFile Get Best Server Node:** Calls the GoFile API.
3. **Log Node:** Displays the result.

A typical chain is **Create Account → Upload File → Get Content → Delete Content**, passing the `token` and IDs with expressions.

### Example Configuration

```json
{
  "zone": "eu"
}
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

#### `Failed to get best server: Failed to get server information`
- **Cause:** GoFile answered with a status other than `ok`.
- **Solution:** Retry later.

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

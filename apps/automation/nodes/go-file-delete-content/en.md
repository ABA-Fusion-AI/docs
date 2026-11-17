---
node_id: "go-file-delete-content"
title: "GoFile Delete Content"
description: "Delete one or several GoFile.io files or folders by content ID (requires the owner's token)."
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
  - go-file-get-best-server
---

<!-- SECTION: overview -->
# GoFile Delete Content

> **Category:** Storage & Files &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Delete files or folders from **GoFile.io**. Several IDs can be deleted at once by separating them with commas. The **token of the owner** is required.

> Deletion is **permanent**.

This node is part of the GoFile family: *Create Account*, *Get Best Server*, *Upload File*, *Get Content*, *Delete Content*.

### Use Cases

- **Cleanup:** Remove temporary files after they were downloaded.
- **Retention:** Delete shared files after a given time.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `contentId` | `string` | Yes* | — | ID of the file or folder, or several IDs separated by commas (`id1,id2`). |
| `token` | `string` | Yes | — | Token of the account that owns the content. |

\* If `contentId` is empty, a string input is used.

### Authentication

Use the `token` returned by **GoFile Create Account**, the `guest_token` returned by **GoFile Upload File**, or the token of your GoFile account (**My Profile → Account token** on gofile.io).

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
| `content_id` | `string` | ID(s) deleted. |
| `message` | `string` | `Content successfully deleted`. |

### Output Example

```json
{
  "status": "ok",
  "content_id": "3f1c2d4e-5a6b-7c8d-9e0f-112233445566",
  "message": "Content successfully deleted"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: GoFile Delete Content
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **GoFile Delete Content Node:** Calls the GoFile API.
3. **Log Node:** Displays the result.

A typical chain is **Create Account → Upload File → Get Content → Delete Content**, passing the `token` and IDs with expressions.

### Example Configuration

```json
{
  "contentId": "{{ input.file_id }}",
  "token": "{{ secrets.GOFILE_TOKEN }}"
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

#### `Failed to delete content: Token is required to delete content`
- **Cause:** `token` is empty.
- **Solution:** Provide the owner's token (for a guest upload, the `guest_token` returned by *Upload File*).

#### `Failed to delete content: Some contents were not deleted: <id>: error-...`
- **Cause:** One of the IDs does not exist or does not belong to the token's account.
- **Solution:** Check the IDs and the token.

#### `Failed to delete content: Unauthorized` / `Forbidden`
- **Cause:** Invalid token, or not the owner.
- **Solution:** Use the owner's token.

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

---
node_id: "go-file-upload-file"
title: "GoFile Upload File"
description: "Download a file from a URL and upload it to GoFile.io, then get its download page."
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
  - go-file-get-content
  - go-file-delete-content
  - go-file-get-best-server
---

<!-- SECTION: overview -->
# GoFile Upload File

> **Category:** Storage & Files &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Upload a file to **GoFile.io** from a public URL. The node downloads the file, sends it to GoFile and returns the **download page** link and the file information.

Without a token, GoFile creates a **guest account** for the upload and returns its `guest_token`: keep it to manage or delete the file later.

This node is part of the GoFile family: *Create Account*, *Get Best Server*, *Upload File*, *Get Content*, *Delete Content*.

### Use Cases

- **File sharing:** Turn a generated report or export into a shareable download link.
- **Backup:** Copy files from another service to GoFile.
- **Large files:** Share files too big for email attachments.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fileUrl` | `string` | Yes* | — | Public URL of the file to upload. The file name is taken from the URL path. |
| `folderId` | `string` | No | — | GoFile folder to upload into (requires the token of the folder's owner). |
| `token` | `string` | No | — | GoFile account token. Without it, a guest account is created. |
| `server` | `string` | No | — | Storage server name (e.g. from *GoFile Get Best Server*). Empty = global upload endpoint. |

\* If `fileUrl` is empty, a string input is used as the URL.

Uploads time out after **2 minutes**; the file is first downloaded in memory, so very large files may fail.

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
| `download_page` | `string` | Public download page (`https://gofile.io/d/<code>`). |
| `code` | `string` | Short code of the folder. |
| `file_id` | `string` | ID of the uploaded file (use it with *Get Content* / *Delete Content*). |
| `file_name` | `string` | File name. |
| `parent_folder` | `string` | ID of the folder containing the file. |
| `size` | `number` | Size in bytes. |
| `md5` | `string` | MD5 checksum. |
| `guest_token` | `string` | Only when no token was given: token of the guest account created. |

### Output Example

```json
{
  "status": "ok",
  "download_page": "https://gofile.io/d/AbC123",
  "code": "AbC123",
  "file_id": "3f1c2d4e-5a6b-7c8d-9e0f-112233445566",
  "file_name": "report.pdf",
  "parent_folder": "9a8b7c6d-5e4f-3a2b-1c0d-ffeeddccbbaa",
  "size": 48213,
  "md5": "5d41402abc4b2a76b9719d911017c592",
  "guest_token": "qWeRtY123456"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: GoFile Upload File
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **GoFile Upload File Node:** Calls the GoFile API.
3. **Log Node:** Displays the result.

A typical chain is **Create Account → Upload File → Get Content → Delete Content**, passing the `token` and IDs with expressions.

### Example Configuration

```json
{
  "fileUrl": "https://example.com/files/report.pdf",
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

#### `Failed to upload file: File URL is required`
- **Cause:** `fileUrl` is empty and the input is not a string.
- **Solution:** Fill in `fileUrl`.

#### `Failed to upload file: Request failed with status code 404` (or another code)
- **Cause:** The file could not be downloaded from `fileUrl` (private, missing or blocked).
- **Solution:** Check that the URL is public and opens in a browser.

#### `Failed to upload file: Upload failed (HTTP 401, GoFile status: error-token)`
- **Cause:** Invalid token.
- **Solution:** Use a valid token, or leave it empty to upload as a guest.

#### `Upload failed (... GoFile status: error-notPremium)` / folder errors
- **Cause:** The `folderId` does not belong to the token's account, or the feature needs a premium account.
- **Solution:** Remove `folderId` or use the owner's token.

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

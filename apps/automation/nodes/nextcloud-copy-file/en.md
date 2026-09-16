---
node_id: "nextcloud-copy-file"
title: "Nextcloud - Copy File"
description: "Copy a file to another location in Nextcloud using WebDAV."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-16"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - file
  - copy
  - storage
related_nodes:
  - nextcloud-move-file
  - nextcloud-delete-file
  - nextcloud-download-file
  - nextcloud-upload-file
  - log
---

<!-- SECTION: header -->
# Nextcloud - Copy File

> **Category:** Productivity | **Subcategory:** File Management | **Type:** Action Node

Copy a file from one location to another in the same Nextcloud account using a WebDAV `COPY` request.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Copy File** node creates a copy of an existing file in Nextcloud. Configure a Nextcloud connection, specify the source and destination paths, and choose whether an existing destination file may be replaced.

The source file remains in its original location. To relocate a file instead, use **Nextcloud - Move File**.

### Key Features

- Copies a file within the connected user's Nextcloud storage.
- Supports source and destination paths relative to the user's file root.
- Optionally replaces an existing file at the destination.
- Returns the HTTP status and response body from Nextcloud.
- Reports connection and WebDAV errors through the error output.

### Use Cases

- Create a backup copy before processing a file.
- Duplicate a template into a working folder.
- Copy generated files into an archive or distribution folder.
- Preserve an original file while preparing a separate version.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | — | Connection containing the Nextcloud base URL, username, and password. |
| `sourcePath` | string | Yes | — | Path of the existing file, relative to the connected user's Nextcloud file root. |
| `destinationPath` | string | Yes | — | Path and filename to assign to the copied file. |
| `overwrite` | boolean | No | `false` | When enabled, allows Nextcloud to replace an existing file at the destination. |

Both file paths must contain at least one character. A leading slash is optional because the node removes leading slashes before building the WebDAV URL.

### Connection

The connection requires:

| Field | Description |
|-------|-------------|
| `baseUrl` | Root URL of the Nextcloud server, such as `https://cloud.example.com`. A trailing slash is removed automatically. |
| `username` | Nextcloud username used in the WebDAV file path and for authentication. |
| `password` | Password or app password used for HTTP Basic authentication. |

For better security, use a Nextcloud app password when available. The connected account must be able to read the source file and write to the destination folder.

### Path Examples

| Purpose | Example |
|---------|---------|
| Source file | `Documents/report.pdf` |
| Destination file | `Archives/report-copy.pdf` |

Paths are relative to the connected user's file area. The destination folder must already exist; this node does not create missing folders.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node performs one fixed operation.

| Operation | Protocol | Method | Description |
|-----------|----------|--------|-------------|
| Copy File | WebDAV | `COPY` | Copies the source file to the destination path in the same Nextcloud account. |

The node sends the destination URL in the WebDAV `Destination` header. Its `Overwrite` header is set to `T` when `overwrite` is enabled and `F` otherwise.

### Overwrite Behavior

- With `overwrite: false`, the request normally fails if a file already exists at the destination.
- With `overwrite: true`, Nextcloud may replace the existing destination file, subject to server permissions and policy.

The node does not compare files, rename conflicts automatically, or delete the source after copying.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action. Incoming data is not used to populate the configuration. |
| `success` | object | Contains the Nextcloud HTTP `status` and response `body`. |
| `error` | runtime error | Receives connection failures and unsuccessful Nextcloud responses. |

### Success Response

On success, the node returns:

```json
{
  "status": 201,
  "body": ""
}
```

The exact status and body are supplied by the Nextcloud server and can vary. A successful WebDAV copy commonly returns an empty body.

### Error Behavior

- Network or connection failures produce an error beginning with `COPY fetch failed` and include the source and destination URLs.
- Non-successful HTTP responses produce an error beginning with `Nextcloud COPY error`, followed by the HTTP status and response body.

Because connection errors can contain server paths and usernames, review error messages before forwarding them to external systems.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Copy a Document

The following configuration copies `test.txt` while preserving the original file:

```json
{
  "connection": {
    "baseUrl": "https://cloud.example.com",
    "username": "your-username",
    "password": "your-app-password"
  },
  "sourcePath": "Documents/test.txt",
  "destinationPath": "Documents/test-copy.txt",
  "overwrite": false
}
```

### Replace an Existing Backup

```json
{
  "connection": {
    "baseUrl": "https://cloud.example.com",
    "username": "your-username",
    "password": "your-app-password"
  },
  "sourcePath": "Reports/latest.pdf",
  "destinationPath": "Backups/latest.pdf",
  "overwrite": true
}
```

Do not place real credentials directly in shared workflow examples or documentation.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The supplied example connects **Manual Trigger → Nextcloud - Copy File → Log**. It copies `Documents/test.txt` to `Documents/test-copy.txt` and sends the success result to the Log node.

```fusion-workflow
src: example.workflow.json
title: Copy a file in Nextcloud
```

### Common Pattern

1. Start the workflow manually or from another trigger.
2. Copy the required file with **Nextcloud - Copy File**.
3. Connect the success output to **Log** or a downstream action.
4. Connect the error output to an error-handling or notification path for production workflows.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | Likely cause | Action |
|-------|--------------|--------|
| Source file is not found | `sourcePath` is incorrect or the file is outside the connected user's storage | Verify the path and filename relative to the user's Nextcloud file root. |
| Destination already exists | `overwrite` is disabled | Enable `overwrite` only if replacing the destination is intended, or choose another destination path. |
| Destination folder is not found | The parent folder does not exist | Create the destination folder before running this node. |
| Authentication fails | Username, password, app password, or base URL is incorrect | Verify the connection and confirm that WebDAV access is enabled. |
| Permission denied | The account cannot read the source or write to the destination | Review file ownership, shares, and folder permissions in Nextcloud. |
| Network request fails | The server is unavailable, its URL is unreachable, or TLS/network settings block access | Check the base URL, server availability, certificate, and network access. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Move File** — Relocate a file instead of retaining the source.
- **Nextcloud - Upload File** — Add a new file to Nextcloud.
- **Nextcloud - Download File** — Retrieve file content from Nextcloud.
- **Nextcloud - Delete File** — Remove a file from Nextcloud.
- **Log** — Inspect the status and response body returned by the copy operation.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-16 | Initial documentation for Nextcloud file copying, overwrite behavior, outputs, workflow integration, and troubleshooting. |

<!-- /SECTION: changelog -->

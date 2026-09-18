---
node_id: "nextcloud-move-file"
title: "Nextcloud - Move File"
description: "Move or rename a file in Nextcloud using WebDAV MOVE."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-18"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - file
  - move
  - cloud-storage
related_nodes:
  - nextcloud-copy-file
  - nextcloud-move-folder
  - nextcloud-download-file
  - nextcloud-delete-file
  - log
---

<!-- SECTION: header -->
# Nextcloud - Move File

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Move or rename a file in the connected user's Nextcloud storage using WebDAV `MOVE`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Move File** node relocates an existing file to a new path in the same Nextcloud account. Use a different folder to move the file, or change its filename to rename it. The original path no longer points to the file after a successful move.

### Use Cases

- Move a processed file into an archive folder.
- Rename a file after a workflow completes.
- Organize files into another existing folder.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | - | Server base URL, username, and password or app password. |
| `sourcePath` | string | Yes | - | Existing file path relative to the connected user's file root; must contain at least one character. |
| `destinationPath` | string | Yes | - | New path and filename relative to the same file root; must contain at least one character. |
| `overwrite` | boolean | No | `false` | Allow the server to replace an existing destination when set to `true`. |

For example, set `sourcePath` to `Documents/report.pdf` and `destinationPath` to `Archives/report.pdf`. To rename a file in place, keep the folder and change only the filename. Leading slashes are removed from both paths when the request URLs are built. Set `connection.baseUrl` to the Nextcloud server root, such as `https://cloud.example.com`.

The connected account needs permission to read or move the source and write to the destination. Ensure the destination folder exists. The schema checks that both paths are non-empty but does not verify that the source is a file or that the destination is safe to replace.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one WebDAV `MOVE` request to the source URL under `/remote.php/dav/files/{username}/`. It sends the destination URL in the `Destination` header and authenticates with the connection's username and password using HTTP Basic authentication. The `Overwrite` header is `F` by default and `T` when `overwrite` is enabled.

When `overwrite` is `false`, an existing destination normally causes the server to reject the move. When it is `true`, the server may replace the destination according to its permissions and policy. The node does not create missing folders, compare file contents, resolve naming conflicts, or retry the request. Incoming workflow data is ignored.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used to populate parameters. |
| `success` | object | Returns `{ "status": number, "body": string }` from the successful server response. |
| `error` | runtime error | Receives network failures and unsuccessful HTTP responses. |

The status and body depend on the server. A successful move may return an empty body.

A network failure produces an error beginning `MOVE fetch failed` and includes both source and destination URLs. An unsuccessful HTTP response produces an error beginning `Nextcloud MOVE error`, followed by the HTTP status and response body. Review these messages before forwarding them externally because they may reveal file paths or usernames.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example Configuration

```json
{
  "connection": {
    "baseUrl": "https://cloud.example.com",
    "username": "your-username",
    "password": "your-app-password"
  },
  "sourcePath": "Documents/report.pdf",
  "destinationPath": "Archives/report.pdf",
  "overwrite": false
}
```

These connection values are placeholders. Do not put real credentials in a shared workflow file. Enable `overwrite` only when replacing the destination is intended.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example contains two **Manual Trigger -> Nextcloud - Move File -> Log** flows. One moves `Readme.md` to `Documents/Readme-copy.md` with the default overwrite setting. The other moves `Readme1.md` to `Documents/Readme--copy.png` with overwrite enabled. Review both paths carefully: changing a filename extension does not convert the file's contents.

```fusion-workflow
src: example.workflow.json
title: Move a file in Nextcloud
```

Replace the example connection values before running it. For production workflows, connect the error output to an error-handling step and confirm the destination before enabling overwrite.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Source file not found | Confirm `sourcePath` relative to the connected user's file root, including spelling and case. |
| Destination already exists | Choose another `destinationPath` or enable `overwrite` only if replacement is intended. |
| Destination folder is missing | Create the parent folder before running the node. |
| Authentication or permission error | Check the connection credentials and the account's access to both locations. |
| File has the wrong type after moving | Review the filename extension; this node moves bytes without converting the file. |
| Network request fails | Verify Nextcloud availability and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Copy File** - Keep the source while creating a second file.
- **Nextcloud - Move Folder** - Relocate a folder.
- **Nextcloud - Download File** - Retrieve file content.
- **Nextcloud - Delete File** - Remove a file.
- **Log** - Inspect the server status and response body.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-18 | Initial documentation for moving or renaming Nextcloud files through WebDAV. |

<!-- /SECTION: changelog -->

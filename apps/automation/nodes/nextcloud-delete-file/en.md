---
node_id: "nextcloud-delete-file"
title: "Nextcloud - Delete File"
description: "Delete a file from Nextcloud using WebDAV DELETE."
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
  - delete
  - cloud-storage
related_nodes:
  - nextcloud-copy-file
  - nextcloud-move-file
  - nextcloud-download-file
  - nextcloud-delete-folder
  - log
---

<!-- SECTION: header -->
# Nextcloud - Delete File

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Delete a file from the connected user's Nextcloud storage using WebDAV `DELETE`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Delete File** node sends a delete request for one configured path. Use it to remove files that are no longer needed after a workflow has finished processing them.

Deletion is a destructive operation. Confirm the target path and the connected account before running it. The node does not create a backup, request confirmation, or verify whether the server can restore the file afterward.

### Use Cases

- Remove a temporary file after it has been processed.
- Clean up an obsolete export or report.
- Delete a file after a successful transfer to another location.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connection` | Nextcloud connection | Yes | Nextcloud server base URL, username, and password or app password. |
| `path` | string | Yes | Path of the file to delete, relative to the connected user's file root; must contain at least one character. |

For example, `Documents/old-report.pdf` identifies a file in the user's `Documents` folder. A leading slash is removed before the URL is built. The connection's `baseUrl` should be the server root, such as `https://cloud.example.com`.

The configured account needs permission to delete the target. The schema requires only a non-empty `path`; it does not confirm that the path identifies a file rather than a folder. Check the target carefully before execution.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node makes one WebDAV `DELETE` request to the configured path under `/remote.php/dav/files/{username}/`. It authenticates with the connection's username and password using HTTP Basic authentication. Incoming workflow data is ignored, so it does not change the configured path.

The node does not perform a separate existence check, move the file to another location, or implement a retry. Any retention or trash-bin behavior is determined by the Nextcloud server, not by this node.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used. |
| `success` | object | Returns `{ "status": number, "body": string }` from a successful server response. |
| `error` | runtime error | Receives network failures and unsuccessful HTTP responses. |

The status and response body depend on the server. A successful deletion may return an empty body.

A network failure produces an error beginning `DELETE fetch failed` and includes the target URL. A non-successful HTTP response produces an error beginning `Nextcloud DELETE error`, followed by the HTTP status and response body. Review these messages before forwarding them to external systems because they may reveal server paths or usernames.

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
  "path": "Documents/old-report.pdf"
}
```

These connection values are placeholders. Do not put real credentials in a shared workflow file.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger → Nextcloud - Delete File → Log**. Review its configured path and replace its connection values before running it: executing the example can delete the target file.

```fusion-workflow
src: example.workflow.json
title: Delete a file from Nextcloud
```

For production workflows, connect the error output to an error-handling step and ensure any preceding transfer or backup has succeeded before deletion.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| File not found | Confirm `path` relative to the connected user's file root and check spelling and case. |
| Permission denied | Verify the connected account can delete the target. |
| Authentication fails | Check the server URL, username, and password or app password. |
| Network request fails | Verify Nextcloud availability and connectivity. |
| Unexpected target | Stop the workflow and review the configured path and connection before retrying. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Copy File** — Preserve a separate copy before deletion.
- **Nextcloud - Move File** — Relocate a file without deleting it outright.
- **Nextcloud - Download File** — Retrieve a file before removing it.
- **Nextcloud - Delete Folder** — Remove a folder.
- **Log** — Inspect the server status and response body.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-16 | Initial documentation for deleting a Nextcloud file through WebDAV. |

<!-- /SECTION: changelog -->

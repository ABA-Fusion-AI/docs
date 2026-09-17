---
node_id: "nextcloud-delete-folder"
title: "Nextcloud - Delete Folder"
description: "Delete a folder or resource from Nextcloud using WebDAV DELETE."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-17"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - folder
  - delete
  - cloud-storage
related_nodes:
  - nextcloud-create-folder
  - nextcloud-copy-folder
  - nextcloud-move-folder
  - nextcloud-delete-file
  - log
---

<!-- SECTION: header -->
# Nextcloud - Delete Folder

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Delete a folder or another resource from the connected user's Nextcloud storage using WebDAV `DELETE`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Delete Folder** node sends a delete request for one configured path. Use it to remove folders that are no longer needed after a workflow has finished processing them.

Deletion is destructive and may remove the folder's contents. Confirm the target path and connected account before running the node. The node does not create a backup, ask for confirmation, or verify whether the server can restore the resource afterward.

### Use Cases

- Remove a temporary working folder after processing.
- Clean up an obsolete project or export directory.
- Delete an archive after its contents have been transferred elsewhere.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connection` | Nextcloud connection | Yes | Nextcloud server base URL, username, and password or app password. |
| `path` | string | Yes | Folder or resource path to delete, relative to the connected user's file root; must contain at least one character. |

For example, `Documents/Old Project` identifies a folder under the user's `Documents` folder. A leading slash is removed when the URL is built. Set the connection's `baseUrl` to the server root, such as `https://cloud.example.com`.

The connected account must have permission to delete the target. The schema only requires a non-empty `path`; it does not verify that the path identifies a folder. Review the target carefully before execution.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one WebDAV `DELETE` request to the configured path under `/remote.php/dav/files/{username}/`. It uses the connection's username and password for HTTP Basic authentication. Incoming workflow data is ignored and does not change the configured path.

The node does not perform a separate existence check, inspect the folder's contents, create a backup, or retry the request. Trash-bin and retention behavior is controlled by the Nextcloud server.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used. |
| `success` | object | Returns `{ "status": number, "body": string }` from a successful server response. |
| `error` | runtime error | Receives network failures and unsuccessful HTTP responses. |

The response status and body depend on the server. A successful deletion may return an empty body.

A network failure produces an error beginning `DELETE fetch failed` and includes the target URL. An unsuccessful HTTP response produces an error beginning `Nextcloud DELETE error`, followed by the HTTP status and response body. Review these errors before forwarding them to external systems because they may reveal server paths or usernames.

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
  "path": "Documents/Old Project"
}
```

These connection values are placeholders. Do not put real credentials in a shared workflow file.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger -> Nextcloud - Delete Folder -> Log** and targets `Documents/Mer`. Review its path and replace its connection values before running it because executing the example can delete that folder and its contents.

```fusion-workflow
src: example.workflow.json
title: Delete a folder from Nextcloud
```

For production workflows, connect the error output to an error-handling step and confirm that any required transfer or backup has completed before deletion.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Folder not found | Confirm `path` relative to the connected user's file root, including spelling and case. |
| Permission denied | Verify that the connected account can delete the target and its contents. |
| Authentication fails | Check the server URL, username, and password or app password. |
| Network request fails | Verify Nextcloud availability and network connectivity. |
| Unexpected target | Stop the workflow and review the configured path and connection before retrying. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Create Folder** - Create a folder.
- **Nextcloud - Copy Folder** - Preserve a separate copy before deletion.
- **Nextcloud - Move Folder** - Relocate a folder.
- **Nextcloud - Delete File** - Remove a single file.
- **Log** - Inspect the server status and response body.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-17 | Initial documentation for deleting a Nextcloud folder or resource through WebDAV. |

<!-- /SECTION: changelog -->

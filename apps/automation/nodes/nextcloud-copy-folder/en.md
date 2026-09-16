---
node_id: "nextcloud-copy-folder"
title: "Nextcloud - Copy Folder"
description: "Copy a folder within Nextcloud using WebDAV COPY."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-16"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - folder
  - copy
  - cloud-storage
related_nodes:
  - nextcloud-copy-file
  - nextcloud-move-folder
  - nextcloud-create-folder
  - nextcloud-delete-folder
  - log
---

<!-- SECTION: header -->
# Nextcloud - Copy Folder

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Create a copy of a folder in the connected user's Nextcloud storage using WebDAV `COPY`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Copy Folder** node copies a source folder to a destination path. The original folder remains in place. Use **Nextcloud - Move Folder** if you need to relocate it instead.

This node makes one WebDAV request per execution. It does not inspect folder contents, create a missing destination parent folder, or verify the copied files afterward.

### Use Cases

- Preserve a working folder before making changes.
- Duplicate a folder of templates for a new project.
- Copy a folder into an archive location.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | — | Server base URL, username, and password or app password. |
| `sourcePath` | string | Yes | — | Existing folder path relative to the connected user's file root. Must not be empty. |
| `destinationPath` | string | Yes | — | Path for the new folder copy, relative to the same file root. Must not be empty. |
| `overwrite` | boolean | No | `false` | Sends WebDAV `Overwrite: T` when enabled, or `Overwrite: F` when disabled. |

For example, set `sourcePath` to `Documents` and `destinationPath` to `Documents-copy`. A leading slash on either path is removed before the request. The connection's `baseUrl` should be the server root, such as `https://cloud.example.com`; one trailing slash is removed when building the request URL.

The connected account needs access to the source folder and permission to write at the destination. Use a Nextcloud app password where possible, and store credentials securely.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends a WebDAV `COPY` request to the source folder under the connected user's `/remote.php/dav/files/` area. The destination URL is supplied in the `Destination` header, and the connection is authenticated with HTTP Basic authentication.

With `overwrite: false`, a destination that already exists may cause the server to reject the request. Enable `overwrite` only when replacing an existing destination is intended. The final behavior, including how an existing folder is handled, depends on the Nextcloud server's WebDAV response and permissions.

The source folder is not deleted. Incoming workflow data does not change the configured paths or overwrite setting.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used by the node. |
| `success` | object | Returns `{ "status": number, "body": string }` from the successful WebDAV response. |
| `error` | runtime error | Receives network failures or unsuccessful HTTP responses. |

The exact success status and response body depend on the server. A network failure raises an error beginning `COPY fetch failed` and includes the source and destination URLs. A non-successful HTTP response raises `Nextcloud COPY error` followed by the status and response body.

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
  "sourcePath": "Documents",
  "destinationPath": "Documents-copy",
  "overwrite": false
}
```

Replace the placeholders with a secure connection. Do not publish real credentials in shared workflows.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied workflow connects **Manual Trigger → Nextcloud - Copy Folder → Log**. It copies `Documents` to `Documents-copy` and logs the success response. Review and replace the connection values in that workflow before using or sharing it.

```fusion-workflow
src: example.workflow.json
title: Copy a folder in Nextcloud
```

Connect the error output to an error-handling step for production workflows.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | Likely cause | Action |
|-------|--------------|--------|
| Source not found | Incorrect `sourcePath` or inaccessible folder | Confirm the path relative to the connected user's file root. |
| Destination already exists | `overwrite` is `false` | Choose a new destination or deliberately enable overwrite. |
| Destination parent is missing | The target parent folder does not exist | Create the parent folder before copying. |
| Authentication or permission error | Invalid credentials or insufficient access | Check the connection and the user's source and destination permissions. |
| Network request fails | Server URL or connectivity problem | Verify the base URL, server availability, and network access. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Copy File** — Copy one file rather than a folder.
- **Nextcloud - Move Folder** — Relocate a folder instead of keeping the source.
- **Nextcloud - Create Folder** — Create a destination parent folder first.
- **Nextcloud - Delete Folder** — Remove a folder when it is no longer needed.
- **Log** — Inspect the returned status and body.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-16 | Initial documentation for copying Nextcloud folders. |

<!-- /SECTION: changelog -->

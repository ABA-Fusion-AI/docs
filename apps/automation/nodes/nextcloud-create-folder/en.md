---
node_id: "nextcloud-create-folder"
title: "Nextcloud - Create Folder"
description: "Create a folder in Nextcloud using WebDAV MKCOL."
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
  - create
  - cloud-storage
related_nodes:
  - nextcloud-copy-folder
  - nextcloud-move-folder
  - nextcloud-delete-folder
  - log
---

<!-- SECTION: header -->
# Nextcloud - Create Folder

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Create a folder in the connected user's Nextcloud storage using WebDAV `MKCOL`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Create Folder** node creates a folder at a specified path. Use it to prepare a destination before copying, moving, or uploading files. The supplied example workflow configures a Nextcloud connection and a folder `path`, then logs the result.

### Use Cases

- Prepare a project folder before adding files.
- Create an archive destination for a workflow.
- Organize output into a dedicated Nextcloud folder.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Description |
|-----------|------|-------------|
| `connection` | Nextcloud connection | Connection to the Nextcloud server, including its base URL, username, and password or app password. |
| `path` | string | Folder path to create, relative to the connected user's file area; for example, `Documents/New Project`. |

Use a Nextcloud app password when possible and keep credentials out of shared workflows. The connected account needs permission to create a folder at the chosen location. Create any missing parent folders first; the example does not demonstrate automatic parent-folder creation.

The supplied source code in this request is for **Nextcloud - Copy Folder**, not this node. It cannot confirm additional `path` validation, input-data behavior, or the exact response format of **Create Folder**.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node's workflow metadata identifies its operation as creating a folder or collection at the specified path using WebDAV `MKCOL`. Unlike **Nextcloud - Copy Folder**, this operation is not configured with `sourcePath`, `destinationPath`, or `overwrite`.

If the destination already exists, its parent is missing, or the account lacks permission, the server may reject the request. Handle the node's error output in production workflows.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Description |
|------|-------------|
| `input` | Starts the action. Whether incoming data can populate `path` is not confirmed by the supplied code. |
| `success` | Result of the folder-creation operation. Its exact shape is not confirmed by the supplied code. |
| `error` | Folder-creation failure, such as an invalid path, permission problem, or server error. |

Connect **Log** to the success output to inspect the actual response in your environment.

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
  "path": "Documents/New Project"
}
```

Replace the placeholders with a secure connection before running the workflow.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger → Nextcloud - Create Folder → Log** and creates a folder under `Documents`. Review and replace its connection values before use or sharing.

```fusion-workflow
src: example.workflow.json
title: Create a folder in Nextcloud
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Folder already exists | Choose a new `path` or verify whether the folder has already been created. |
| Parent folder is missing | Create the parent folder before running this node. |
| Authentication fails | Verify the server URL, username, and password or app password. |
| Permission denied | Confirm the account can create folders at the chosen location. |
| Server or network failure | Check Nextcloud availability and connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Copy Folder** — Duplicate an existing folder.
- **Nextcloud - Move Folder** — Relocate an existing folder.
- **Nextcloud - Delete Folder** — Remove a folder.
- **Log** — Inspect the creation result.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-16 | Initial documentation based on the Create Folder workflow metadata and example. |

<!-- /SECTION: changelog -->

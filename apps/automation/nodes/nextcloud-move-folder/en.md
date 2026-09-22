---
node_id: "nextcloud-move-folder"
title: "Nextcloud - Move Folder"
description: "Move or rename a folder in Nextcloud using WebDAV MOVE."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-22"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - folder
  - move
  - cloud-storage
related_nodes:
  - nextcloud-copy-folder
  - nextcloud-move-file
  - nextcloud-create-folder
  - nextcloud-delete-folder
  - log
---

<!-- SECTION: header -->
# Nextcloud - Move Folder

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Move or rename a folder in the connected user's Nextcloud storage using WebDAV `MOVE`. The node's current editor label is **Nextcloud - Move/Move Folder**.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Move Folder** node requests a move from an existing folder path to a destination path in the same Nextcloud account. Change the parent path to relocate the folder, or change the final folder name to rename it in place.

The node sends one request and returns the server's status and response body. It does not inspect the folder's contents or verify the result with a second request.

### Use Cases

- Move a completed project folder to an archive location.
- Rename a folder after processing its contents.
- Organize folders under an existing destination parent.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | - | Nextcloud base URL, username, and password or app password. |
| `sourcePath` | string | Yes | - | Existing folder path relative to the connected user's file root. Must contain at least one character. |
| `destinationPath` | string | Yes | - | Complete destination path, including the folder's new name, relative to the same file root. Must contain at least one character. |
| `overwrite` | boolean | No | `false` | Sends `Overwrite: T` when enabled and `Overwrite: F` otherwise. |

For example, use `Documents-copy` as the source and `Documents-moved` as the destination to rename a folder in place. To request a move under another parent, use a destination such as `Archives/Documents-moved`.

Set `connection.baseUrl` to the Nextcloud installation's base URL, such as `https://cloud.example.com`. The node appends `/remote.php/dav/files/{username}/` itself. Supply ordinary path text: each path segment is URL-encoded when building the request.

The schema checks that both path strings are non-empty. It does not check that the source is a folder, create the destination parent, or validate permissions before sending the request.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node builds source and destination URLs within the connected user's file area and sends a WebDAV `MOVE` request to the source URL. It provides:

- `Authorization`: HTTP Basic authentication using the connection's username and password.
- `Destination`: the complete destination URL.
- `Overwrite`: `T` when `overwrite` is `true`, otherwise `F`.

Use `overwrite: true` only when replacement of an existing destination is intended. The server determines whether the move is allowed and reports the result in its response.

Incoming workflow data is ignored; the action uses the configured connection and paths. The node does not retry failed requests or create missing parent folders.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used by the handler. |
| `success` | object | Returns `{ "status": number, "body": string }` for a successful HTTP response. |
| `error` | runtime error | Reports request failures and unsuccessful HTTP responses. |

`status` is the HTTP response status. `body` is the response read as text, without JSON parsing; it can be empty. A response is treated as successful when `res.ok` is true.

If the fetch request fails, the node throws an error in this format:

```text
Nextcloud MOVE failed from "<sourcePath>" to "<destinationPath>" (HTTP status: unavailable; response body: unavailable): <network error>
```

For an unsuccessful HTTP response, it throws:

```text
Nextcloud MOVE failed from "<sourcePath>" to "<destinationPath>" (HTTP status: <status>; response body: <body>)
```

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
  "sourcePath": "Documents-copy",
  "destinationPath": "Documents-moved",
  "overwrite": false
}
```

Replace the connection placeholders with your Nextcloud connection. This configuration requests a rename from `Documents-copy` to `Documents-moved` within the user's file root.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger -> Nextcloud - Move/Move Folder -> Log**. It moves `Documents-copy` to `Documents-moved` and passes the successful response to Log. The example omits `overwrite`, so the schema default of `false` applies.

```fusion-workflow
src: example.workflow.json
title: Move a folder in Nextcloud
```

Replace the example connection values and confirm the source and destination paths before running the workflow. Connect the error output to an error-handling step to handle failed moves.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Source folder cannot be found | Check `sourcePath` relative to the connected user's file root, including spelling and case. |
| Destination already exists | Choose a different destination or enable `overwrite` if replacement is intended. |
| Destination parent is missing | Create the parent folder before running this node. |
| Authentication or permission failure | Check the connection credentials and the account's access to the source and destination. |
| HTTP request is rejected | Inspect the status and response body included in the error message. |
| HTTP status is unavailable | Check the base URL, server availability, and network connectivity; inspect the appended network error. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Copy Folder** - Create a folder copy while keeping the source.
- **Nextcloud - Move File** - Move or rename an individual file.
- **Nextcloud - Create Folder** - Create a destination parent folder.
- **Nextcloud - Delete Folder** - Remove a folder.
- **Log** - Inspect the returned status and response body.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-22 | Initial documentation for moving or renaming Nextcloud folders through WebDAV. |

<!-- /SECTION: changelog -->

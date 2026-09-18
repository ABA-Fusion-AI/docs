---
node_id: "nextcloud-list-folders"
title: "Nextcloud - List Folders"
description: "List files and folders at a Nextcloud path using WebDAV."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-18"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - files
  - folders
  - cloud-storage
related_nodes:
  - nextcloud-create-folder
  - nextcloud-copy-folder
  - nextcloud-move-folder
  - nextcloud-download-file
  - log
---

<!-- SECTION: header -->
# Nextcloud - List Folders

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

List files and folders at a path in the connected user's Nextcloud storage.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - List Folders** node retrieves WebDAV metadata for a configured path and its immediate children. Despite its name, the result can contain both files and folders. Each item indicates whether it is a directory.

### Use Cases

- Inspect the contents of a Nextcloud folder.
- Find files available for a later download or move step.
- Check folder names before organizing or cleaning up storage.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | - | Server base URL, username, and password or app password. |
| `path` | string | No | `/` | Path to list relative to the connected user's file root. |

Omit `path` to use `/`, the connected user's file root. For example, `Documents/Projects` selects a folder under `Documents`. The node removes one leading slash from the configured path when constructing the request URL. Set `connection.baseUrl` to the server root, such as `https://cloud.example.com`.

The connected account must be able to read the selected location. The schema accepts any string for `path`; it does not verify that the path exists or identifies a folder.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node first sends an HTTP `GET` request to `/remote.php/dav/files/{username}/{path}`. If that request succeeds, it sends a WebDAV `PROPFIND` request to the same URL with `Depth: 1` and `Prefer: return-minimal`. Both requests use the connection's username and password for HTTP Basic authentication.

The `PROPFIND` response is parsed as XML. The node looks for response entries and returns metadata for the selected resource and any immediate children included by the server. It does not recursively list deeper levels or filter the result to folders only. Incoming workflow data is ignored, and the node does not retry failed requests.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used. |
| `success` | object | Usually returns `{ "items": [...] }` with the listed resources. If XML processing throws, returns `{ "xml": "...", "parseError": "..." }` instead. |
| `error` | runtime error | Receives network failures or unsuccessful HTTP responses from either request. |

Each item may include `href`, `displayName`, `isDirectory`, `size`, and `lastModified`. `isDirectory` is a boolean; the other fields can be absent when Nextcloud does not provide them. `size` is converted to a number when present. The node also attempts a text-based fallback if XML parsing produces no items.

A failed initial request produces an error beginning `fetch failed when calling` or `Nextcloud WebDAV error`. A failed listing request produces an error beginning `PROPFIND fetch failed when calling` or `Nextcloud PROPFIND error`. Network errors include the request URL; HTTP errors include the status and server response text. Review errors before forwarding them externally because they may reveal paths or usernames.

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
  "path": "Documents/Projects"
}
```

Omit `path` to list the file root. These connection values are placeholders; do not put real credentials in a shared workflow file.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger -> Nextcloud - List Folders -> Log**. It omits `path`, so the node uses the default `/` and logs the returned listing. Replace the connection placeholders before running it.

```fusion-workflow
src: example.workflow.json
title: List files and folders in Nextcloud
```

For production workflows, connect the error output to an error-handling step. Filter the returned `items` by `isDirectory` if the next step needs folders only.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Path not found | Confirm `path` relative to the connected user's file root, including spelling and case. |
| Permission denied | Verify the connected account can read the selected location. |
| Authentication fails | Check the server URL, username, and password or app password. |
| Files appear in the result | Filter `items` using `isDirectory`; the node lists both files and folders. |
| Expected children are missing | Confirm the selected path and inspect the server's WebDAV response; the node lists only one level. |
| XML returned instead of items | Inspect `parseError` and `xml` to diagnose an unexpected response format. |
| Network request fails | Verify Nextcloud availability and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Create Folder** - Add a folder at a chosen path.
- **Nextcloud - Copy Folder** - Duplicate a folder.
- **Nextcloud - Move Folder** - Relocate a folder.
- **Nextcloud - Download File** - Retrieve a listed file.
- **Log** - Inspect the returned items during testing.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-18 | Initial documentation for listing Nextcloud files and folders through WebDAV. |

<!-- /SECTION: changelog -->

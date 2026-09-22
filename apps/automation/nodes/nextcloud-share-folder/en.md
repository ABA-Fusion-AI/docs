---
node_id: "nextcloud-share-folder"
title: "Nextcloud - Share Folder"
description: "Create a user, group, or public share for a Nextcloud folder using the OCS Share API."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-22"
author: "Fusion Team"
tags:
  - nextcloud
  - folder
  - sharing
  - ocs
  - cloud-storage
related_nodes:
  - nextcloud-get-user
  - nextcloud-share-file
  - nextcloud-move-folder
  - log
---

<!-- SECTION: header -->
# Nextcloud - Share Folder

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Create a share for a configured Nextcloud folder using the OCS Share API.

The handler sends the configured folder path to Nextcloud without checking the resource type locally.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Share Folder** node submits a share creation request for a folder path in the connected Nextcloud account. Choose a user share, a group share, or a public share. The server determines whether the requested share and permissions are allowed.

The node returns the response as raw text. It does not extract a share URL, parse the OCS response, or check the API-level status inside the response body.

### Use Cases

- Share a project folder with a specified Nextcloud user.
- Request access for a Nextcloud group.
- Create a public share for a folder when allowed by the server.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | - | Nextcloud base URL, username, and password or app password. |
| `path` | string | Yes | - | Folder path to share in the connected account. Must contain at least one character. |
| `shareType` | enum | No | `public` | One of `user`, `group`, or `public`. |
| `shareWith` | string | No | - | Target user or group identifier for a recipient-based share. Sent only when non-empty. |
| `permissions` | number | No | `1` | Numeric permission value sent to Nextcloud. Sent only when truthy; `0` is omitted. |

Set `connection.baseUrl` to the Nextcloud installation's base URL, such as `https://cloud.example.com`. The handler appends the OCS shares endpoint itself.

For user or group shares, provide the intended recipient in `shareWith`. The schema does not require this field conditionally, so the server handles requests with a missing or invalid recipient. The schema also does not restrict `permissions` to integers, a range, or a set of supported values.

The default share type is `public`. Set `shareType` explicitly when the share should target a user or group.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one HTTP `POST` request to:

```text
{baseUrl}/ocs/v2.php/apps/files_sharing/api/v1/shares
```

The request uses HTTP Basic authentication with the connection credentials, `OCS-APIREQUEST: true`, and `Content-Type: application/x-www-form-urlencoded`.

The form includes `path` and maps `shareType` as follows:

| Configured value | Submitted value |
|------------------|-----------------|
| `user` | `0` |
| `group` | `1` |
| `public` | `3` |

A non-empty `shareWith` is included regardless of the selected share type. A truthy `permissions` value is converted to a string and included. The path is submitted through `URLSearchParams`; the handler does not trim it or remove leading slashes.

Incoming workflow data is ignored. The node does not look up recipients, check for existing shares, retry failed requests, or expose parameters for share passwords and expiration dates.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used by the handler. |
| `success` | object | Returns `{ "raw": string }` when the HTTP response is successful. |
| `error` | runtime error | Reports fetch failures and unsuccessful HTTP responses. |

`raw` contains the full response body as text. The node does not request a specific response format or parse the returned body. Add a downstream parsing and validation step if the workflow needs a share identifier, URL, or OCS status.

A successful HTTP response alone does not establish that the OCS response reports a successful share creation: this handler checks only `res.ok`.

If the fetch request fails, the node throws:

```text
Share POST failed for <url>: <network error>
```

For an unsuccessful HTTP response, it throws:

```text
Nextcloud Share API error: <status> <response text>
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
  "path": "Documents",
  "shareType": "user",
  "shareWith": "recipient-username",
  "permissions": 1
}
```

Replace the connection and recipient placeholders with your Nextcloud values. For a group share, set `shareType` to `group` and supply the group identifier. For a public share, set `shareType` to `public` and omit `shareWith`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example connects **Manual Trigger -> Nextcloud - Share Folder -> Log**. It requests a public share for `Documents`. It omits `shareWith` and `permissions`, so no recipient is sent and the permission default of `1` applies.

```fusion-workflow
src: example.workflow.json
title: Share a folder in Nextcloud
```

Replace the example connection and folder path before running it. Confirm that a public share is intended, or configure a user or group share with its recipient identifier. Log receives the raw response object. Inspect the OCS response to confirm share creation, and connect the error output to an error-handling step.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| Path cannot be shared | Confirm the path exists in the connected account and the account can share it. |
| User or group share fails | Check `shareType` and provide the correct recipient identifier in `shareWith`. |
| Public share is rejected | Check whether the server allows public shares and requires options this node does not expose. |
| Permissions are rejected or unexpected | Check the numeric value accepted by the server. A configured value of `0` is not sent. |
| Success output contains an API error | Inspect the OCS status inside `raw`; the node checks HTTP success only. |
| No direct share URL in the output | Parse `raw` and inspect the returned share data; the node does not extract fields. |
| Authentication or HTTP error | Check the connection credentials and inspect the status and response text in the error. |
| Network request fails | Check the base URL, server availability, and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Get User** - Retrieve details for a Nextcloud user.
- **Nextcloud - Share File** - Create a share for an individual file.
- **Nextcloud - Move Folder** - Move or rename a folder.
- **Log** - Inspect the raw share response.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-22 | Initial documentation for creating Nextcloud folder shares through the OCS Share API. |

<!-- /SECTION: changelog -->

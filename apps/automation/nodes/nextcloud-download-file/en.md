---
node_id: "nextcloud-download-file"
title: "Nextcloud - Download File"
description: "Download a file from Nextcloud using WebDAV GET."
category: "Storage & Files"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-09-17"
author: "Fusion Team"
tags:
  - nextcloud
  - webdav
  - file
  - download
  - cloud-storage
related_nodes:
  - nextcloud-upload-file
  - nextcloud-copy-file
  - nextcloud-move-file
  - nextcloud-delete-file
  - log
---

<!-- SECTION: header -->
# Nextcloud - Download File

> **Category:** Storage & Files | **Subcategory:** Cloud Drives | **Type:** Action Node

Download a file from the connected user's Nextcloud storage using WebDAV `GET`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nextcloud - Download File** node retrieves one file from a configured path. It returns the downloaded content as a binary buffer by default or as a Base64-encoded string when `asBase64` is enabled.

### Use Cases

- Retrieve a document for processing in a workflow.
- Download an image or attachment for transfer to another service.
- Encode file content as Base64 for a text-based API or storage step.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connection` | Nextcloud connection | Yes | - | Nextcloud server base URL, username, and password or app password. |
| `path` | string | Yes | - | File path relative to the connected user's file root; must contain at least one character. |
| `asBase64` | boolean | No | `false` | Return the file body as a Base64 string instead of a binary buffer. |

For example, `Documents/report.pdf` identifies a file in the user's `Documents` folder. A leading slash is removed when the request URL is built. Set `connection.baseUrl` to the server root, such as `https://cloud.example.com`.

The connected account must have permission to read the target file. The schema only requires a non-empty `path`; it does not verify that the path exists or refers to a file.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operation -->
## Operation

The node sends one WebDAV `GET` request to the configured path under `/remote.php/dav/files/{username}/`. It authenticates with the connection's username and password using HTTP Basic authentication. Incoming workflow data is ignored and does not change the configured path.

After a successful response, the node reads the entire response into memory as an `ArrayBuffer`. It then converts the content to a Node.js `Buffer`, or to a Base64 string when `asBase64` is `true`. The node does not stream the response, preserve response headers, or retry a failed request.

Base64 output is text-safe but is larger than the original binary content. Large downloads also consume memory because the complete file is buffered before the node returns.

<!-- /SECTION: operation -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Type | Description |
|------|------|-------------|
| `input` | `unknown` | Starts the action; its value is not used. |
| `success` | object | Returns `{ "status": number, "body": Buffer \| string }`. The body is a binary `Buffer` by default and a Base64 string when `asBase64` is enabled. |
| `error` | runtime error | Receives network failures and unsuccessful HTTP responses. |

The `status` value is the successful HTTP response status. The result does not include the file name, content type, content length, or other response headers.

A network failure produces an error beginning `GET fetch failed` and includes the target URL. An unsuccessful HTTP response produces an error beginning `Nextcloud GET error`, followed by the HTTP status and response text. Review errors before forwarding them externally because they may reveal server paths or usernames.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example Configuration

Return the downloaded file as Base64:

```json
{
  "connection": {
    "baseUrl": "https://cloud.example.com",
    "username": "your-username",
    "password": "your-app-password"
  },
  "path": "Documents/report.pdf",
  "asBase64": true
}
```

Set `asBase64` to `false`, or omit it, to receive a binary buffer. These connection values are placeholders; do not put real credentials in a shared workflow file.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

The supplied example contains flows that connect **Manual Trigger -> Nextcloud - Download File -> Log** for `Documents/test.txt`, including a Base64 configuration. Replace all connection values before running or sharing the example, and remove any credentials already stored in it.

```fusion-workflow
src: example.workflow.json
title: Download a file from Nextcloud
```

Choose the output format expected by the downstream node. Use Base64 for systems that require text, and use the binary buffer when the next step accepts binary content.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Issue | What to check |
|-------|---------------|
| File not found | Confirm `path` relative to the connected user's file root, including spelling and case. |
| Permission denied | Verify that the connected account can read the target file. |
| Authentication fails | Check the server URL, username, and password or app password. |
| Downstream node cannot use the body | Set `asBase64` to the format that the receiving node expects. |
| Large file uses too much memory | Reduce the file size or use another download method that supports streaming. |
| Network request fails | Verify Nextcloud availability and network connectivity. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Nextcloud - Upload File** - Send a file to Nextcloud.
- **Nextcloud - Copy File** - Duplicate a file within Nextcloud.
- **Nextcloud - Move File** - Relocate a file within Nextcloud.
- **Nextcloud - Delete File** - Remove a file after processing.
- **Log** - Inspect the returned status and body during testing.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-17 | Initial documentation for downloading Nextcloud files through WebDAV. |

<!-- /SECTION: changelog -->

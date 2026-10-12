---
node_id: "kobotoolbox-file"
title: "KoboToolbox File"
description: "Manage media attachments, map layers, and files for KoboToolbox forms."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-12"
author: "Fusion Team"
tags: [kobotoolbox, files, attachments, media, peer-only]
related_nodes: ["kobotoolbox-form", "kobotoolbox-submission", "kobotoolbox-hook", "kobotoolbox"]
---

<!-- SECTION: overview -->
# KoboToolbox File

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **KoboToolbox File** node provides complete file and attachment management for forms within KoboToolbox. Use this node to list form media, upload reference files (such as lookup CSVs or image prompts), retrieve file metadata, or delete obsolete attachments.

### Key Capabilities
- **List Files (`getMany`)**: Retrieve all media files attached to a form, with filtering by `file_type` (`form_media` or `map_layer`).
- **Get File (`get`)**: Fetch metadata and properties for an individual file by UID.
- **Create File (`create`)**: Upload reference files using base64 encoded content or external redirect URLs.
- **Delete File (`delete`)**: Remove files from the form project.
<!-- /SECTION: overview -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Configuration Parameters
- **Operation (`operation`)**: `getMany`, `get`, `create`, or `delete`.
- **API Token (`apiToken`)**: KoboToolbox API authentication token.
- **Base URL (`baseUrl`)**: Server URL (Default: `https://kf.kobotoolbox.org`).
- **Form ID (`formId`)**: Form asset UID.
- **File ID (`fileId`)**: File UID (required for `get` and `delete`).
- **File Type (`fileType`)**: `form_media` or `map_layer`.
- **Filename (`filename`)**: Name of the media file (e.g. `choices.csv` or `logo.png`).
- **Content Base64 (`contentBase64`)**: Base64 encoded payload when creating a file.
- **Redirect URL (`redirectUrl`)**: Remote URL when linking external assets.

### Outputs
- **Success Output**: Emits the file metadata, list of files, or deletion confirmation.
- **Error Output**: Emits error message and code when remote operations fail.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Upload and Manage Form Media
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

- Ensure files uploaded do not contain malicious executable payloads.
- Store API tokens securely in Fusion credential management.
<!-- /SECTION: security -->

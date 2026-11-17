---
node_id: "go-file-get-content"
title: "GoFile Get Content"
description: "Get the details of a GoFile.io file or folder (name, size, children, links) by content ID."
category: "Storage & Files"
subcategory: "File Sharing"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - gofile
  - file-sharing
  - storage
  - upload
related_nodes:
  - go-file-create-account
  - go-file-upload-file
  - go-file-delete-content
  - go-file-get-best-server
---

<!-- SECTION: overview -->
# GoFile Get Content

> **Category:** Storage & Files &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read the details of a **GoFile.io** file or folder: name, type, creation date, visibility and, for a folder, the list of its contents with size, MIME type and links.

This node is part of the GoFile family: *Create Account*, *Get Best Server*, *Upload File*, *Get Content*, *Delete Content*.

### Use Cases

- **File inventory:** List the files of a GoFile folder.
- **Verification:** Check that an upload is complete (size, MD5).
- **Download links:** Get the direct links of the files of a folder.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `contentId` | `string` | Yes* | — | ID of the file or folder (e.g. `file_id` / `parent_folder` from *Upload File*). |
| `token` | `string` | No | — | GoFile account token. |

\* If `contentId` is empty, a string input is used.

> **Premium account:** GoFile restricts this endpoint (`GET /contents/{id}`) to **premium** accounts. With a free or guest token, the node fails with `This GoFile endpoint requires a premium account`.

### Authentication

Use the `token` returned by **GoFile Create Account**, the `guest_token` returned by **GoFile Upload File**, or the token of your GoFile account (**My Profile → Account token** on gofile.io).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` / `string` | Triggers the node. Upstream values can be used through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Result of the operation (see below). |
| `error` | `Error` | Emitted on missing fields or GoFile errors. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | `string` | `ok` on success. |
| `is_owner` | `boolean` | Whether the token owns the content. |
| `id / type / name / code` | `string` | Content identity (`type` is `file` or `folder`). |
| `parent_folder` | `string` | Parent folder ID. |
| `create_time` | `string` | Creation date (ISO). |
| `is_public` | `boolean` | Public visibility. |
| `children_count / children_ids` | `number / array` | Children of a folder. |
| `contents[]` | `array` | `id`, `type`, `name`, `size`, `create_time`, `download_count`, `md5`, `mimetype`, `link`, `thumbnail`. |

### Output Example

```json
{
  "status": "ok",
  "is_owner": true,
  "id": "9a8b7c6d-5e4f-3a2b-1c0d-ffeeddccbbaa",
  "type": "folder",
  "name": "AbC123",
  "code": "AbC123",
  "parent_folder": "1234abcd-...",
  "create_time": "2026-11-17T15:20:00.000Z",
  "is_public": true,
  "children_count": 1,
  "children_ids": [
    "3f1c2d4e-5a6b-7c8d-9e0f-112233445566"
  ],
  "contents": [
    {
      "id": "3f1c2d4e-5a6b-7c8d-9e0f-112233445566",
      "type": "file",
      "name": "report.pdf",
      "size": 48213,
      "mimetype": "application/pdf",
      "link": "https://store1.gofile.io/download/web/.../report.pdf"
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: GoFile Get Content
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **GoFile Get Content Node:** Calls the GoFile API.
3. **Log Node:** Displays the result.

A typical chain is **Create Account → Upload File → Get Content → Delete Content**, passing the `token` and IDs with expressions.

### Example Configuration

```json
{
  "contentId": "{{ input.parent_folder }}",
  "token": "{{ secrets.GOFILE_TOKEN }}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store GoFile tokens in Fusion's **Secrets** vault. Do not commit hardcoded tokens into version control.

- A GoFile token gives full access to the account's files: treat it like a password.
- Files uploaded to GoFile are reachable by anyone who has the download link.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to get content: This GoFile endpoint requires a premium account`
- **Cause:** Free and guest accounts cannot use this endpoint.
- **Solution:** Use a premium GoFile token.

#### `Failed to get content: Not Found: Content or resource not found`
- **Cause:** Unknown ID, or the content was deleted.
- **Solution:** Check the ID.

#### `Failed to get content: Unauthorized: Invalid or missing token`
- **Cause:** Private content without a valid token.
- **Solution:** Provide the owner's token.

#### `... Rate Limited: Too many requests` / `Server Error: GoFile.io is experiencing issues`
- **Cause:** GoFile rate limit or temporary outage. The node already retried up to 3 times.
- **Solution:** Wait and run the workflow again.

#### `... Request timeout after 60000ms`
- **Cause:** GoFile did not answer in time.
- **Solution:** Retry later.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation |

<!-- /SECTION: changelog -->

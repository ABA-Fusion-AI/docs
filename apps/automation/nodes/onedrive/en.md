---
node_id: "onedrive"
title: "OneDrive"
description: "Upload, download, get, search, copy, rename, share and delete OneDrive files, and create, list, search, rename, share and delete folders through Microsoft Graph."
category: "integrations"
subcategory: "Cloud Drives"
version: "1.0.0"
language: "en"
last_updated: "2026-11-23"
author: "Fusion Team"
tags:
  - integration
  - onedrive
  - microsoft
  - files
  - storage
  - microsoft-graph
related_nodes:
  - google-drive-file
  - google-drive-folder
  - outlook-mail
---

<!-- SECTION: overview -->
# OneDrive

> **Category:** Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage files and folders in the **OneDrive** of the signed-in user through the **Microsoft Graph API v1.0** (`/me/drive`). Choose a **type** (`file` or `folder`), then an operation. The node offers **14 operations**:

| Type | Operations |
|------|------------|
| `file` | `upload`, `download`, `get`, `search`, `copy`, `rename`, `share`, `delete` |
| `folder` | `create`, `getChildren`, `search`, `rename`, `share`, `delete` |

### Use Cases

- **Reports:** Upload a generated CSV, PDF or text report to a OneDrive folder.
- **File pipelines:** Download a file, process it in the workflow, and upload the result.
- **Sharing:** Create a view or edit link and send it by email or chat.
- **Organisation:** Create dated folders, rename or clean up old files.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiToken` | `string` | Yes | Microsoft Graph **access token** with `Files.ReadWrite` (or `Files.ReadWrite.All`). Sent as `Authorization: Bearer <token>`. |

- The node always works on the drive of the **token owner** (`/me/drive`), for personal (OneDrive.com) and work/school (OneDrive for Business) accounts.
- For tests you can copy a token from **Graph Explorer → Access token** (sign in, consent to `Files.ReadWrite`). It expires after about **1 hour**. Paste the whole token, without the `Bearer` prefix.

### Item IDs

Most operations take an **item ID** (file or folder). Get IDs from the output of `upload`, `create`, `getChildren`, `get` or `search` (`id` field), e.g. `671065E290326728!sb1f27fd456f84d18b3ea6cf7638b0c46`.

> Use **`root`** as the parent ID to work at the top level of the drive (`create`, `upload`, `getChildren`, `copy`).

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | `enum` | Yes | `file` or `folder`. |
| `fileOperation` | `enum` | When `type` = `file` | `copy`, `delete`, `download`, `get`, `rename`, `search`, `share`, `upload`. |
| `folderOperation` | `enum` | When `type` = `folder` | `create`, `delete`, `getChildren`, `rename`, `search`, `share`. |

#### File operations

| Operation | Parameters | Notes |
|-----------|------------|-------|
| `upload` | `fileUploadParentId` (required), `fileUploadFileName` (required), `fileUploadContent` | Content: plain text (uploaded as UTF-8) or **base64** (decoded to binary). If `fileUploadContent` is empty, the node uses `fileContent`, `content` or `file` from the input data, including the output of `download`. An existing file with the same name is **replaced**. Simple upload: up to **250 MB**. |
| `download` | `fileDownloadItemId` (required) | Returns the content as **base64** with its content type. |
| `get` | `fileGetItemId` (required), `fileGetSelect`, `fileGetExpand` | Metadata. `fileGetSelect` limits the fields (e.g. `id,name,size`), `fileGetExpand` adds relations (e.g. `thumbnails`). |
| `search` | `fileSearchQuery` (required), `fileSearchParentId` | Searches names and content in the whole drive, or under a folder. |
| `copy` | `fileCopyItemId` (required), `fileCopyParentId`, `fileCopyName` | Destination folder (default `root`) and new name. The copy runs **in the background**. |
| `rename` | `fileRenameItemId` (required), `fileRenameNewName` (required) | Keep the extension in the new name. |
| `share` | `fileShareItemId` (required), `fileShareType`, `fileShareScope` | Creates a sharing link. Type `view` (default), `edit` or `embed`; scope `anonymous` (default, anyone with the link) or `organization`. |
| `delete` | `fileDeleteItemId` (required) | Moves the file to the OneDrive **recycle bin**. |

#### Folder operations

| Operation | Parameters | Notes |
|-----------|------------|-------|
| `create` | `folderCreateParentId` (required, `root` for the top level), `folderCreateName` (required) | If the name already exists, OneDrive adds a number (`Reports 1`). |
| `getChildren` | `folderGetChildrenItemId` (required, `root` allowed), `folderGetChildrenTop`, `folderGetChildrenSkip` | Lists the files and sub-folders. `top` limits the number of items, `skip` drops the first ones. |
| `search` | `folderSearchQuery` (required), `folderSearchParentId` | Same search as files (returns files and folders that match). |
| `rename` | `folderRenameItemId` (required), `folderRenameNewName` (required) | — |
| `share` | `folderShareItemId` (required), `folderShareType`, `folderShareScope` | Same options as file `share`. |
| `delete` | `folderDeleteItemId` (required) | Deletes the folder **and everything in it** (recycle bin). |

> `embed` links are only available for personal accounts, and `organization` links only for work/school accounts.

> Fields are shown only for the selected type and operation.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. For `upload`, it can carry the content in `fileContent`, `content` or `file` (e.g. the output of a `download`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `string` | The Microsoft Graph response, or a message for operations without a response body (`copy`, `delete`). |
| `error` | `Error` | `OneDrive API error: <Graph message>`, or a missing parameter (`Item ID is required for ...`). |

### Output Examples

#### `upload` / `get` / `rename` / folder `create`

```json
{
  "id": "671065E290326728!sbfe6d98e8af644879492f1335d14d713",
  "name": "report.txt",
  "size": 31,
  "createdDateTime": "2026-11-23T16:14:50Z",
  "lastModifiedDateTime": "2026-11-23T16:14:50Z",
  "webUrl": "https://onedrive.live.com?cid=671065E290326728&id=671065E290326728!sbfe6d...",
  "parentReference": { "id": "671065E290326728!sb1f27fd456f84d18b3ea6cf7638b0c46", "path": "/drive/root:/FusionTest" },
  "file": { "mimeType": "text/plain" },
  "@microsoft.graph.downloadUrl": "https://my.microsoftpersonalcontent.com/..."
}
```

#### `download`

```json
{
  "file": {
    "content": "SGVsbG8gZnJvbSBGdXNpb24gT25lRHJpdmUgbm9kZQ==",
    "contentType": "text/plain",
    "extension": "txt",
    "filename": "file.txt"
  }
}
```

> `filename` is generic (`file.<extension>`), not the original file name. Use `get` if you need the real name.

#### `getChildren` / `search`

```json
{
  "value": [
    { "id": "671065E290326728!sbfe6d...", "name": "report.txt", "size": 31, "file": { "mimeType": "text/plain" } },
    { "id": "671065E290326728!sa12b...", "name": "Archive", "folder": { "childCount": 3 } }
  ]
}
```

#### `share`

```json
{
  "id": "dc2785bc-685e-465e-951d-6c480a8876bd",
  "roles": ["read"],
  "hasPassword": false,
  "link": {
    "type": "view",
    "scope": "anonymous",
    "webUrl": "https://1drv.ms/t/c/671065e290326728/IQCO2ea_9oqHRJSS8TNdFNcT..."
  }
}
```

#### `copy` / `delete`

```text
File copy initiated successfully
File deleted successfully
Folder deleted successfully
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use OneDrive in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **OneDrive Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Create a folder at the root

```json
{
  "apiToken": "{{ secrets.MS_GRAPH_TOKEN }}",
  "type": "folder",
  "folderOperation": "create",
  "folderCreateParentId": "root",
  "folderCreateName": "Reports"
}
```

#### Upload a text file

```json
{
  "type": "file",
  "fileOperation": "upload",
  "fileUploadParentId": "671065E290326728!sb1f27fd456f84d18b3ea6cf7638b0c46",
  "fileUploadFileName": "report.txt",
  "fileUploadContent": "Daily report: 42 orders processed."
}
```

#### List a folder

```json
{
  "type": "folder",
  "folderOperation": "getChildren",
  "folderGetChildrenItemId": "root",
  "folderGetChildrenTop": 20
}
```

#### Share a file (read-only link)

```json
{
  "type": "file",
  "fileOperation": "share",
  "fileShareItemId": "671065E290326728!sbfe6d98e8af644879492f1335d14d713",
  "fileShareType": "view",
  "fileShareScope": "anonymous"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token in Fusion's **Secrets** vault. Do not commit it into version control.

- `anonymous` links can be opened by **anyone** who has the URL: prefer `view` links and `organization` scope for internal files.
- `delete` on a folder removes all its content: protect it from repeated triggers.
- The token gives access to the whole drive of its owner: use a dedicated account for automation when possible.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `OneDrive API error: Protocol 'Bearer' failed to validate because The token could not be read.`
- **Cause:** The token is incomplete (truncated copy) or contains extra text.
- **Solution:** Copy the whole token again with the copy button (no `Bearer` prefix, no quotes).

#### `OneDrive API error: AuthenticationException` / `InvalidAuthenticationToken`
- **Cause:** The token expired (Graph Explorer tokens last about 1 hour) or was replaced.
- **Solution:** Get a new token and paste it in `apiToken`.

#### `OneDrive API error: Access denied` (403)
- **Cause:** The token does not have `Files.ReadWrite`.
- **Solution:** Consent to `Files.ReadWrite` and get a new token.

#### `Parent ID is required for create` / `Parent ID is required for upload`
- **Cause:** No parent folder.
- **Solution:** Use a folder ID, or `root` for the top level.

#### `OneDrive API error: Item not found` (404)
- **Cause:** Wrong or deleted item ID.
- **Solution:** Get a fresh ID with `getChildren` or `search`.

#### `search` returns no result for a new file
- **Cause:** OneDrive indexes new files with a delay.
- **Solution:** Wait a few minutes, or use `getChildren` on the parent folder.

#### `File content is required for upload`
- **Cause:** `fileUploadContent` is empty and the input has no `fileContent`, `content` or `file`.
- **Solution:** Fill in `fileUploadContent` or pass the content in the input.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-23 | Initial documentation: 14 file and folder operations, authentication, item IDs, output examples, troubleshooting |

<!-- /SECTION: changelog -->

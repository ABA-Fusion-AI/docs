---
node_id: "sftp"
title: "SFTP"
description: "Performs SFTP actions with support for uploading/downloading folders and large directories."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-06"
author: "Fusion Team"
tags: [sftp, files, transfer, directories, integration, peer-only]
related_nodes: [function]
---

<!-- SECTION: overview -->
# SFTP

> **Category:** Peer-only Integrations | **Type:** Action Node

Connect to an SFTP server to upload files or folders, download folders, list directory entries in batches, or delete a remote file. The node uses `ssh2-sftp-client` and opens a new connection for each execution.

### Use Cases

- Upload locally generated reports to an SFTP server.
- Download remote directories for downstream processing.
- Retrieve directory metadata in grouped results.
- Delete a specified remote file.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | Yes | None | SFTP server hostname or address. |
| `port` | `number` | No | `22` | SFTP connection port. |
| `username` | `string` | Yes | None | Account used to connect. |
| `password` | `string` | No in schema | None | Password passed to the SFTP client. |
| `privateKey` | `string` | No in schema | None | Private key contents passed to the client, rather than a path that this node reads. |
| `action` | `enum` | Yes | None | `upload-folder`, `upload`, `download-folder`, `download`, `list`, or `delete`. |
| `remotePath` | `string` | Yes | None | Remote source, target directory, or file path, depending on the action. |
| `localPath` | `string` | All upload/download actions at runtime | None | Local source or destination on the machine running the node. |
| `maxFiles` | `number` | No | `20000` | Maximum matching entries returned by `list`. Does not limit transfers. |
| `filter` | `string` | No | None | JavaScript regular expression pattern applied to entry names for `list`, directory `upload`, and `download`. |
| `batchSize` | `number` | No | `100` | Number of entries grouped into each returned listing batch. |

The schema declares no expression metadata or conditional field visibility. Numeric fields have no integer or range constraints; use a valid port and positive integers for `maxFiles` and `batchSize`.

Password and private key are both optional in the schema. Provide authentication accepted by the server; the handler does not enforce a particular method or expose a private-key passphrase field. Local paths refer to the node's runtime filesystem, which may differ from your own computer.

### Actions

| Action | Behavior | Path requirements | Filter |
|--------|----------|-------------------|--------|
| `upload-folder` | Calls the client's `uploadDir(localPath, remotePath)` directory-transfer method. | Local source directory and remote destination directory | Ignored |
| `upload` | Uploads a local file, or processes the immediate entries of a local directory. | Local source path; remote file path for a single file or target directory for directory entries | Applied to directory entry names only |
| `download-folder` | Calls the client's `downloadDir(remotePath, localPath)` directory-transfer method. | Remote source directory and local destination directory | Ignored |
| `download` | Lists a remote directory, downloads regular files, and recursively visits matching subdirectories. | Remote source directory and local destination directory | Applied to file and directory names at every level |
| `list` | Lists one remote directory and groups matching entries into returned batches. | Remote directory | Applied to entry names |
| `delete` | Calls the client's `delete(remotePath)` file-deletion method. | Remote file | Ignored |

### Upload Details

For `upload`, the node inspects `localPath` with `fs.statSync`. A file is sent directly to `remotePath`; the filter is compiled but does not select or exclude that single file.

For a local directory, the node reads immediate entry names and uploads matching entries sequentially to `<remotePath>/<name>`. It does not recurse or check that each entry is a regular file. A matching subdirectory is passed to `put` and may cause the operation to fail. Use `upload-folder` for directory transfers containing subdirectories. The custom `upload` helper does not create the remote target directory.

### Download Details

The `download` helper always treats `remotePath` as a directory and calls `list` on it; it does not implement a direct single-file download action. It creates the local destination directory recursively when absent, downloads regular files, and recursively processes directories. Other remote entry types are skipped.

The filter applies to directory names as well as file names. If a directory does not match, the node skips that directory and all of its contents. For example, a pattern matching only `.csv` filenames can prevent traversal into ordinary subdirectories.

### Listing and Batching

`list` first fetches the complete immediate directory listing from the server. It then filters entries, groups them into batches, and returns all batches together. It does not stream batches to downstream steps or recursively list subdirectories. Batching and `maxFiles` do not limit the initial listing fetched into memory.

`maxFiles` counts matching entries, including directories and special entry types. The limit is checked after adding each entry. With zero or a negative limit, a non-empty matching listing can still return one entry. A fractional positive limit can return the next whole entry above that value. Use positive integers for predictable results.

`maxReached` means the returned count reached the configured limit; it does not prove that additional matching entries remain. No continuation cursor or automatic pagination is provided.

### Regular Expression Filters

Filters are compiled with `new RegExp(filter)` without a separate flags parameter and matched against names rather than full paths. Omit the filter or use an empty string to disable filtering. An invalid non-empty pattern raises a regular expression error in actions that compile it.

In JSON configuration, a literal dot must be escaped as `\\.`. For example, `"\\.csv$"` matches names ending in `.csv`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

Incoming workflow data has type `unknown` and is not read by the handler. All connection settings, actions, and paths come from node configuration.

### Outputs

| Action | Returned result |
|--------|-----------------|
| `upload-folder` | The client's `uploadDir` return value, passed through unchanged. |
| `download-folder` | The client's `downloadDir` return value, passed through unchanged. |
| `upload` with a file | `Uploaded single file <localPath>` |
| `upload` with a directory | `Uploaded <count> files from folder <localPath>` |
| `download` | `Downloaded <count> files from <remotePath> to <localPath>` |
| `delete` | The client's `delete` return value, passed through unchanged. |
| `list` | An object containing counts and batched entry metadata. |

The `download` count describes processed entries at the current directory level, including directories. Counts from recursive calls are not added to the parent result, so it is not a total of all downloaded files.

### Listing Fields

| Field | Description |
|-------|-------------|
| `totalReturned` | Number of matching entries included in the result. |
| `maxReached` | Whether `totalReturned >= maxFiles`. |
| `batchCount` | Number of arrays in `batches`. |
| `batches` | Array of entry arrays. Each entry contains `name`, `size`, `modifyTime`, and `rights`, copied from the client listing. |

Listing entries do not include their remote type or full path. The node does not convert modification times or permission fields.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

These JSON objects represent node parameters. Replace credential placeholders with protected values and choose paths accessible to the runtime and SFTP account.

### List CSV Entries

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "password": "YOUR_PASSWORD",
  "action": "list",
  "remotePath": "/incoming",
  "filter": "\\.csv$",
  "maxFiles": 1000,
  "batchSize": 100
}
```

### Upload a Single File

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "privateKey": "YOUR_PRIVATE_KEY_CONTENTS",
  "action": "upload",
  "localPath": "C:/automation/reports/daily.csv",
  "remotePath": "/reports/daily.csv"
}
```

### Upload CSV Files from One Directory Level

Ensure the remote target directory exists and the selected local entries are files.

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "password": "YOUR_PASSWORD",
  "action": "upload",
  "localPath": "C:/automation/reports",
  "remotePath": "/reports",
  "filter": "\\.csv$"
}
```

### Upload a Folder

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "password": "YOUR_PASSWORD",
  "action": "upload-folder",
  "localPath": "C:/automation/export",
  "remotePath": "/exports"
}
```

### Download a Folder with the Client Directory Method

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "password": "YOUR_PASSWORD",
  "action": "download-folder",
  "remotePath": "/incoming",
  "localPath": "C:/automation/incoming"
}
```

### Recursively Download a Remote Directory

Omit the filter to allow traversal of all regular directories and files.

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "password": "YOUR_PASSWORD",
  "action": "download",
  "remotePath": "/incoming",
  "localPath": "C:/automation/incoming"
}
```

### Delete a Remote File

```json
{
  "host": "sftp.example.com",
  "username": "automation",
  "password": "YOUR_PASSWORD",
  "action": "delete",
  "remotePath": "/incoming/processed.csv"
}
```

### Illustrative Listing Output

```json
{
  "totalReturned": 1,
  "maxReached": false,
  "batchCount": 1,
  "batches": [
    [
      {
        "name": "daily.csv",
        "size": 2048,
        "modifyTime": 1791244800000,
        "rights": {
          "user": "rw",
          "group": "r",
          "other": ""
        }
      }
    ]
  ]
}
```

### Workflow Patterns

- Report generation → SFTP (`upload`).
- Scheduled trigger → SFTP (`download-folder`) → local file processing.
- SFTP (`list`) → Function to inspect or flatten returned batches.

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Cause | Resolution |
|------------------|-------|------------|
| `localPath is required for upload` | An upload action has no non-empty local path. | Supply the local source path. |
| `localPath is required for download` | A download action has no non-empty local path. | Supply the local destination directory. |
| `Unknown SFTP action` | An unsupported action reached the handler. | Select one of the six schema-supported actions. |
| Connection or authentication failure | Host, port, username, or authentication settings are rejected or unreachable. | Check connectivity and credentials accepted by the server. |
| Invalid regular expression | The filter cannot be compiled. | Correct the pattern and its JSON escaping. |
| Upload fails on a subdirectory | Directory `upload` passes a selected directory entry to `put`. | Use `upload-folder`, or select only regular files. |
| Download fails for a remote file path | The `download` helper expects a directory to list. | Supply a remote directory; this implementation has no direct single-file download action. |
| Nested files are missing | The download filter excluded a parent directory. | Omit the filter or make it match the required directory names too. |
| Transfer filter appears ignored | `upload-folder` and `download-folder` do not use `filter`. | Choose the helper action whose documented filtering behavior matches your needs. |
| File or permission error | A local/remote path is missing, inaccessible, or unsuitable for the action. | Check paths and filesystem permissions for the runtime and SFTP account. |

The connection is established before action-specific validation. After a successful connection, the handler calls `sftp.end()` in a `finally` block, including when an action fails. Connection failures occur before that block. A failure from `end()` can replace an action's result or error.

Errors are propagated without a custom wrapper, retries, or rollback. If a multi-file action fails partway through, earlier transfers can remain completed. The `stop()` method performs no cleanup or cancellation, and the node exposes no timeout parameter.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security

Store passwords and private key contents in protected configuration. Do not include real credentials in exported examples or logs. Use an SFTP account with permissions appropriate to the required paths and actions.

The supplied node does not expose a host-key verification setting, key passphrase, or error-redaction mechanism. File operations run with the runtime's local filesystem permissions and the remote account's server permissions.

<!-- /SECTION: security -->

---

<!-- SECTION: related -->
## Related

- [Function](../function/en.md) – Process listing batches and prepare downstream file-processing data.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-06 | Documentation generated from the supplied implementation, covering transfers, filtering, listing batches, and connection lifecycle. |

<!-- /SECTION: changelog -->

---
node_id: "s3-folder"
title: "S3 Compatible - Folder"
description: "Prefix (folder-like) operations for S3-compatible storage"
category: "integrations"
subcategory: "storage"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - s3
  - folder
  - prefix
  - minio
related_nodes:
  - s3-bucket
  - s3-file
---

<!-- SECTION: header -->

# S3 Compatible - Folder

> **Category:** Integrations | **Type:** Action Node

Manage folder-like prefixes with `list`, `create`, and `delete`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **S3 Compatible - Folder** node treats prefixes as folders and helps list prefix entries, create marker folders, or recursively delete all objects under a folder path.

### Key Features

- List subfolders via delimiter and prefix
- Create folder marker object (`folderPath/`)
- Delete folder contents in batches

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter           | Type      | Required | Default                | Description                        |
| ------------------- | --------- | -------- | ---------------------- | ---------------------------------- |
| `endpoint`          | `string`  | ❌ No    | —                      | Custom S3 endpoint URL             |
| `forcePathStyle`    | `boolean` | ❌ No    | —                      | Enables path-style requests        |
| `profile`           | `string`  | ❌ No    | —                      | AWS shared credentials profile     |
| `region`            | `string`  | ❌ No    | `us-east-1` (fallback) | Region used by SDK                 |
| `accessKeyId`       | `string`  | ❌ No    | —                      | Access key ID                      |
| `secretAccessKey`   | `string`  | ❌ No    | —                      | Secret access key                  |
| `sessionToken`      | `string`  | ❌ No    | —                      | Session token                      |
| `operation`         | `enum`    | ❌ No    | `list`                 | One of: `list`, `create`, `delete` |
| `bucketName`        | `string`  | ✅ Yes   | —                      | Target bucket                      |
| `folderPath`        | `string`  | ✅ Yes\* | —                      | Folder-like prefix path            |
| `delimiter`         | `string`  | ❌ No    | `/`                    | Delimiter for `list`               |
| `maxKeys`           | `number`  | ❌ No    | `1000`                 | Max keys for `list`                |
| `continuationToken` | `string`  | ❌ No    | —                      | Token for paginated `list`         |
| `batchSize`         | `number`  | ❌ No    | `1000`                 | Batch size for `delete` listing    |

\* Required for `create` and `delete`.

### Operation Behavior

| Operation | Behavior                                            | Output                                            |
| --------- | --------------------------------------------------- | ------------------------------------------------- |
| `list`    | List prefix groups (`CommonPrefixes`)               | `{ folders, isTruncated, nextContinuationToken }` |
| `create`  | Create zero-byte folder marker                      | `{ bucket, folder, result }`                      |
| `delete`  | Recursively list+delete objects under folder prefix | `{ deleted, errors }`                             |

<!-- /SECTION: configuration -->

---

<!-- SECTION: examples -->

## Examples

### List folders

```json
{
  "operation": "list",
  "bucketName": "media",
  "folderPath": "images",
  "delimiter": "/"
}
```

### Delete a folder recursively

```json
{
  "operation": "delete",
  "bucketName": "media",
  "folderPath": "tmp/uploads",
  "batchSize": 500
}
```

<!-- /SECTION: examples -->

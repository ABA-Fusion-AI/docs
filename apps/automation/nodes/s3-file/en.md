---
node_id: "s3-file"
title: "S3 Compatible - File"
description: "File operations for S3-compatible providers"
category: "integrations"
subcategory: "storage"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - s3
  - file
  - minio
  - spaces
related_nodes:
  - s3-bucket
  - s3-folder
---

<!-- SECTION: header -->

# S3 Compatible - File

> **Category:** Integrations | **Type:** Action Node

Run file operations for S3-compatible services: `upload`, `download`, `delete`, `list`, and `copy`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **S3 Compatible - File** node handles object-level operations across S3-compatible platforms and supports both local file-path and direct-body uploads.

### Key Features

- Upload from local file or inline body
- Download to local path or return base64 content
- Delete multiple keys in one request
- List objects with pagination token
- Copy objects across buckets/keys

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter           | Type       | Required | Default                | Description                                                |
| ------------------- | ---------- | -------- | ---------------------- | ---------------------------------------------------------- |
| `endpoint`          | `string`   | ❌ No    | —                      | Custom S3 endpoint URL                                     |
| `forcePathStyle`    | `boolean`  | ❌ No    | —                      | Enables path-style requests                                |
| `profile`           | `string`   | ❌ No    | —                      | AWS shared credentials profile                             |
| `region`            | `string`   | ❌ No    | `us-east-1` (fallback) | Region used by SDK                                         |
| `accessKeyId`       | `string`   | ❌ No    | —                      | Access key ID                                              |
| `secretAccessKey`   | `string`   | ❌ No    | —                      | Secret access key                                          |
| `sessionToken`      | `string`   | ❌ No    | —                      | Session token                                              |
| `operation`         | `enum`     | ❌ No    | `upload`               | One of: `upload`, `download`, `delete`, `list`, `copy`     |
| `bucketName`        | `string`   | ✅ Yes\* | —                      | Required for `upload`, `download`, `delete`, `list`        |
| `key`               | `string`   | ✅ Yes\* | —                      | Required for `upload` and `download`                       |
| `filePath`          | `string`   | ❌ No    | —                      | Local path to read during `upload`                         |
| `body`              | `any`      | ❌ No    | —                      | Upload body; if string looks base64, decoded before upload |
| `contentType`       | `string`   | ❌ No    | —                      | Upload content type                                        |
| `saveToPath`        | `string`   | ❌ No    | —                      | Local destination path for `download`                      |
| `prefix`            | `string`   | ❌ No    | —                      | Prefix filter for `list`                                   |
| `maxKeys`           | `number`   | ❌ No    | —                      | Max keys for `list`                                        |
| `continuationToken` | `string`   | ❌ No    | —                      | Pagination token for `list`                                |
| `keys`              | `string[]` | ✅ Yes\* | —                      | Required for `delete` (multi-key delete)                   |
| `sourceBucket`      | `string`   | ✅ Yes\* | —                      | Required for `copy`                                        |
| `sourceKey`         | `string`   | ✅ Yes\* | —                      | Required for `copy`                                        |
| `destinationBucket` | `string`   | ✅ Yes\* | —                      | Required for `copy`                                        |
| `destinationKey`    | `string`   | ✅ Yes\* | —                      | Required for `copy`                                        |

\* Required only for the operations indicated.

### Operation Behavior

| Operation  | Behavior                                           | Output                                                                   |
| ---------- | -------------------------------------------------- | ------------------------------------------------------------------------ |
| `upload`   | `PutObject` using `filePath` or `body`             | `{ bucket, key, result }`                                                |
| `download` | `GetObject`; save to file or return base64         | `{ bucket, key, savedTo }` or `{ bucket, key, contentBase64 }`           |
| `delete`   | `DeleteObjects` for provided `keys`                | `{ deleted, errors }`                                                    |
| `list`     | `ListObjectsV2` with optional prefix/token         | `{ objects, isTruncated, nextContinuationToken }`                        |
| `copy`     | `CopyObject` from source bucket/key to destination | `{ sourceBucket, sourceKey, destinationBucket, destinationKey, result }` |

<!-- /SECTION: configuration -->

---

<!-- SECTION: examples -->

## Examples

### Upload from local file

```json
{
  "operation": "upload",
  "bucketName": "assets",
  "key": "images/logo.png",
  "filePath": "/tmp/logo.png",
  "contentType": "image/png"
}
```

### Download and return content

```json
{
  "operation": "download",
  "bucketName": "assets",
  "key": "images/logo.png"
}
```

### Copy object

```json
{
  "operation": "copy",
  "sourceBucket": "assets",
  "sourceKey": "images/logo.png",
  "destinationBucket": "archive",
  "destinationKey": "images/logo.png"
}
```

<!-- /SECTION: examples -->

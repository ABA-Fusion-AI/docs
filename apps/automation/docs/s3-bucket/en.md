---
node_id: "s3-bucket"
title: "S3 Compatible - Bucket"
description: "Bucket operations for S3-compatible providers"
category: "integrations"
subcategory: "storage"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - s3
  - minio
  - wasabi
  - spaces
related_nodes:
  - s3-file
  - s3-folder
---

<!-- SECTION: header -->

# S3 Compatible - Bucket

> **Category:** Integrations | **Type:** Action Node

Manage buckets on S3-compatible services using `list`, `create`, `delete`, and `search`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **S3 Compatible - Bucket** node works with providers such as MinIO, Wasabi, and DigitalOcean Spaces by allowing custom endpoint and path-style settings.

### Key Features

- List all buckets
- Create or delete a bucket
- Search buckets by substring query
- Support custom endpoint + credential profile

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter         | Type      | Required | Default                | Description                                  |
| ----------------- | --------- | -------- | ---------------------- | -------------------------------------------- |
| `endpoint`        | `string`  | ❌ No    | —                      | Custom S3 endpoint URL                       |
| `forcePathStyle`  | `boolean` | ❌ No    | —                      | Enables path-style requests                  |
| `profile`         | `string`  | ❌ No    | —                      | AWS shared credentials profile               |
| `region`          | `string`  | ❌ No    | `us-east-1` (fallback) | Region used by SDK                           |
| `accessKeyId`     | `string`  | ❌ No    | —                      | Access key ID                                |
| `secretAccessKey` | `string`  | ❌ No    | —                      | Secret access key                            |
| `sessionToken`    | `string`  | ❌ No    | —                      | Session token                                |
| `operation`       | `enum`    | ❌ No    | `list`                 | One of: `list`, `create`, `delete`, `search` |
| `bucketName`      | `string`  | ✅ Yes\* | —                      | Required for `create` and `delete`           |
| `acl`             | `string`  | ❌ No    | —                      | ACL for `create`                             |
| `query`           | `string`  | ❌ No    | —                      | Substring filter used in `search`            |

\* Required only for the listed operations.

### Operation Behavior

| Operation | Behavior                           | Output                                  |
| --------- | ---------------------------------- | --------------------------------------- |
| `list`    | List all buckets                   | `{ buckets: [{ name, creationDate }] }` |
| `create`  | Create bucket with optional ACL    | `{ bucket, result }`                    |
| `delete`  | Delete bucket                      | `{ bucket, result }`                    |
| `search`  | List and filter buckets by `query` | `{ buckets }`                           |

<!-- /SECTION: configuration -->

---

<!-- SECTION: examples -->

## Examples

### List MinIO buckets

```json
{
  "endpoint": "http://localhost:9000",
  "forcePathStyle": true,
  "accessKeyId": "minioadmin",
  "secretAccessKey": "minioadmin",
  "operation": "list"
}
```

### Search buckets

```json
{
  "operation": "search",
  "query": "backup"
}
```

<!-- /SECTION: examples -->

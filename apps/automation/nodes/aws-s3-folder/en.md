---
node_id: "aws-s3-folder"
title: "AWS S3 - Folder"
description: "Manage S3 prefixes as folders: list, create, and delete"
category: "integrations"
subcategory: "aws"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - aws
  - s3
  - folder
  - prefix
related_nodes:
  - aws-s3-file
  - aws-s3-bucket
  - aws-s3-trigger
---

<!-- SECTION: header -->

# AWS S3 - Folder

> **Category:** Integrations | **Type:** Action Node

Perform prefix-level operations in S3: `list`, `create`, and `delete`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AWS S3 - Folder** node treats S3 prefixes as folders. It can list objects under a prefix, create a folder marker object, or delete all objects recursively under a prefix.

### Key Features

- List objects under a prefix with pagination support
- Create folder marker (`prefix/`) as zero-byte object
- Delete all objects under a prefix in 1000-item batches

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter         | Type     | Required | Default                | Description                        |
| ----------------- | -------- | -------- | ---------------------- | ---------------------------------- |
| `accessKeyId`     | `string` | ❌ No    | —                      | AWS access key ID                  |
| `secretAccessKey` | `string` | ❌ No    | —                      | AWS secret access key              |
| `sessionToken`    | `string` | ❌ No    | —                      | AWS session token                  |
| `region`          | `string` | ❌ No    | `us-east-1` (fallback) | AWS region                         |
| `operation`       | `enum`   | ❌ No    | `list`                 | One of: `list`, `create`, `delete` |
| `bucket`          | `string` | ✅ Yes   | —                      | Target bucket                      |
| `prefix`          | `string` | ✅ Yes   | —                      | Folder-like prefix to manage       |
| `pageSize`        | `number` | ❌ No    | —                      | `list` operation max keys per page |
| `pageToken`       | `string` | ❌ No    | —                      | `list` continuation token          |

### Operation Behavior

| Operation | Behavior                                                                | Output                                   |
| --------- | ----------------------------------------------------------------------- | ---------------------------------------- |
| `list`    | `ListObjectsV2` with `Bucket`, `Prefix`, `MaxKeys`, `ContinuationToken` | `{ objects, commonPrefixes, nextToken }` |
| `create`  | `PutObject` for marker key (`prefix/`)                                  | AWS SDK response object                  |
| `delete`  | Lists all keys under prefix and deletes in batches of 1000              | `{ deleted: number }`                    |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input   | Type  | Description            |
| ------- | ----- | ---------------------- |
| `input` | `any` | Optional upstream data |

### Outputs

| Output   | Type    | Description                  |
| -------- | ------- | ---------------------------- |
| `output` | `any`   | Operation result             |
| `error`  | `Error` | Emitted on operation failure |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### List a folder prefix

```json
{
  "operation": "list",
  "bucket": "my-app-bucket",
  "prefix": "invoices/2026/",
  "pageSize": 100
}
```

### Create a folder marker

```json
{
  "operation": "create",
  "bucket": "my-app-bucket",
  "prefix": "invoices/2026"
}
```

### Delete everything under a prefix

```json
{
  "operation": "delete",
  "bucket": "my-app-bucket",
  "prefix": "tmp/uploads/"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Prefix not deleted completely

Ensure the IAM role can run both `s3:ListBucket` and `s3:DeleteObject` on all matching keys.

#### Folder create not visible

S3 has no true folders. The node creates a marker object ending with `/`.

#### Pagination confusion

Use the returned `nextToken` as the next `pageToken` to continue listing.

<!-- /SECTION: troubleshooting -->

---
node_id: "aws-s3-file"
title: "AWS S3 - File"
description: "Upload, download, list, inspect, and delete files in Amazon S3"
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
  - file
  - storage
related_nodes:
  - aws-s3-folder
  - aws-s3-bucket
  - aws-s3-trigger
---

<!-- SECTION: header -->

# AWS S3 - File

> **Category:** Integrations | **Type:** Action Node

Run file-level S3 operations: `upload`, `download`, `delete`, `get`, and `list`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AWS S3 - File** node connects to Amazon S3 and executes a selected operation against a bucket/object key.

### Key Features

- Upload objects with custom content (`body`)
- Download object content (optional body return placeholder)
- Fetch object metadata (`HeadObject`)
- Delete a single object
- List objects by prefix

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter         | Type      | Required | Default                | Description                                                        |
| ----------------- | --------- | -------- | ---------------------- | ------------------------------------------------------------------ |
| `accessKeyId`     | `string`  | ❌ No    | —                      | AWS access key ID. If omitted, runtime AWS credentials are used    |
| `secretAccessKey` | `string`  | ❌ No    | —                      | AWS secret access key (used with `accessKeyId`)                    |
| `sessionToken`    | `string`  | ❌ No    | —                      | AWS session token for temporary credentials                        |
| `region`          | `string`  | ❌ No    | `us-east-1` (fallback) | AWS region                                                         |
| `operation`       | `enum`    | ❌ No    | `upload`               | One of: `upload`, `download`, `delete`, `get`, `list`              |
| `bucket`          | `string`  | ✅ Yes\* | —                      | Target S3 bucket (\*required for all operations)                   |
| `key`             | `string`  | ✅ Yes\* | —                      | Object key (\*required for `upload`, `download`, `delete`, `get`)  |
| `body`            | `any`     | ❌ No    | —                      | Upload body/content (used by `upload`)                             |
| `returnBody`      | `boolean` | ❌ No    | `false`                | For `download`/`get`, returns placeholder body object when enabled |
| `prefix`          | `string`  | ❌ No    | —                      | Filter prefix for `list`                                           |
| `maxKeys`         | `number`  | ❌ No    | —                      | Maximum objects returned by `list`                                 |

### Operation Behavior

| Operation  | Behavior                         | Output                                                               |
| ---------- | -------------------------------- | -------------------------------------------------------------------- |
| `upload`   | `PutObject` to `bucket/key`      | AWS SDK response object                                              |
| `download` | `GetObject` from `bucket/key`    | `{ ok: true }` or `{ body: "<binary data>" }` when `returnBody=true` |
| `delete`   | `DeleteObject` on `bucket/key`   | `{ ok: true }`                                                       |
| `get`      | `HeadObject` on `bucket/key`     | Object metadata response                                             |
| `list`     | `ListObjectsV2` on bucket/prefix | Array of object entries (`Contents`)                                 |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input   | Type  | Description                                                   |
| ------- | ----- | ------------------------------------------------------------- |
| `input` | `any` | Optional upstream data (not required for operation execution) |

### Outputs

| Output   | Type    | Description                     |
| -------- | ------- | ------------------------------- |
| `output` | `any`   | Result of selected operation    |
| `error`  | `Error` | Emitted when AWS SDK call fails |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Upload a JSON file

```json
{
  "operation": "upload",
  "region": "us-east-1",
  "bucket": "my-app-bucket",
  "key": "exports/users.json",
  "body": "{\"users\":[{\"id\":1}]}"
}
```

### List objects in a prefix

```json
{
  "operation": "list",
  "bucket": "my-app-bucket",
  "prefix": "exports/",
  "maxKeys": 200
}
```

### Get object metadata

```json
{
  "operation": "get",
  "bucket": "my-app-bucket",
  "key": "exports/users.json"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Access denied

Ensure IAM credentials include the needed S3 actions (`s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, `s3:ListBucket`, `s3:HeadObject`).

#### Signature mismatch or auth errors

Verify `region`, access keys, and session token are correct and not expired.

#### Empty list result

Check the `prefix` value and ensure objects exist under that prefix.

<!-- /SECTION: troubleshooting -->

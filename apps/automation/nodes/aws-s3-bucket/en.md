---
node_id: "aws-s3-bucket"
title: "AWS S3 - Bucket"
description: "Manage S3 buckets: list, create, delete, and inspect"
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
  - bucket
  - storage
related_nodes:
  - aws-s3-file
  - aws-s3-folder
  - aws-s3-trigger
---

<!-- SECTION: header -->

# AWS S3 - Bucket

> **Category:** Integrations | **Type:** Action Node

Run bucket-level operations in Amazon S3: `list`, `create`, `delete`, and `get`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AWS S3 - Bucket** node is used for bucket administration flows, including discovery, provisioning, cleanup, and metadata inspection.

### Key Features

- List available buckets
- Create a bucket (optional ACL parameter)
- Delete a bucket
- Check bucket accessibility and region (`get` operation)

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter         | Type     | Required | Default                | Description                                            |
| ----------------- | -------- | -------- | ---------------------- | ------------------------------------------------------ |
| `accessKeyId`     | `string` | ❌ No    | —                      | AWS access key ID                                      |
| `secretAccessKey` | `string` | ❌ No    | —                      | AWS secret access key                                  |
| `sessionToken`    | `string` | ❌ No    | —                      | AWS session token                                      |
| `region`          | `string` | ❌ No    | `us-east-1` (fallback) | AWS region                                             |
| `operation`       | `enum`   | ❌ No    | `list`                 | One of: `list`, `create`, `delete`, `get`              |
| `bucket`          | `string` | ✅ Yes\* | —                      | Bucket name (\*required for `create`, `delete`, `get`) |
| `acl`             | `string` | ❌ No    | —                      | Canned ACL passed during `create`                      |

### Operation Behavior

| Operation | Behavior                                        | Output                  |
| --------- | ----------------------------------------------- | ----------------------- |
| `list`    | `ListBuckets` request                           | Array of bucket entries |
| `create`  | `CreateBucket` with `Bucket` and optional `ACL` | AWS SDK response object |
| `delete`  | `DeleteBucket` for selected bucket              | `{ ok: true }`          |
| `get`     | `HeadBucket` then `GetBucketLocation`           | `{ bucket, location }`  |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input   | Type  | Description            |
| ------- | ----- | ---------------------- |
| `input` | `any` | Optional upstream data |

### Outputs

| Output   | Type    | Description                         |
| -------- | ------- | ----------------------------------- |
| `output` | `any`   | Result of selected bucket operation |
| `error`  | `Error` | Emitted when operation fails        |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### List buckets

```json
{
  "operation": "list",
  "region": "us-east-1"
}
```

### Create a bucket

```json
{
  "operation": "create",
  "region": "us-east-1",
  "bucket": "my-fusion-bucket"
}
```

### Get bucket location

```json
{
  "operation": "get",
  "bucket": "my-fusion-bucket"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Bucket already exists

S3 bucket names are globally unique. Choose a unique name.

#### Delete bucket fails

S3 only allows deleting empty buckets. Remove all objects first.

#### Permission denied

Ensure IAM permissions include required actions such as `s3:ListAllMyBuckets`, `s3:CreateBucket`, `s3:DeleteBucket`, `s3:HeadBucket`, and `s3:GetBucketLocation`.

<!-- /SECTION: troubleshooting -->

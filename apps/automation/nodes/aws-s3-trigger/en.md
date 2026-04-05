---
node_id: "aws-s3-trigger"
title: "AWS S3 - New Object Trigger"
description: "Poll an S3 bucket/prefix and emit newly detected objects"
category: "triggers"
subcategory: "aws"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - aws
  - s3
  - polling
  - new-object
related_nodes:
  - aws-s3-file
  - aws-s3-folder
  - aws-s3-bucket
---

<!-- SECTION: header -->

# AWS S3 - New Object Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls S3 at a fixed interval and emits only objects not seen before by this node instance.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AWS S3 - New Object Trigger** node scans a bucket (optionally scoped by prefix), keeps an internal cache of known object keys, and emits newly discovered objects.

### Key Features

- Poll-based S3 trigger (no webhook required)
- Prefix filtering
- Warm start mode via `emitOnStart`
- Seen-key cache persisted in workflow variables
- Pagination support for large buckets

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter         | Type      | Required | Default                | Description                                         |
| ----------------- | --------- | -------- | ---------------------- | --------------------------------------------------- |
| `accessKeyId`     | `string`  | ❌ No    | —                      | AWS access key ID                                   |
| `secretAccessKey` | `string`  | ❌ No    | —                      | AWS secret access key                               |
| `sessionToken`    | `string`  | ❌ No    | —                      | AWS session token                                   |
| `region`          | `string`  | ❌ No    | `us-east-1` (fallback) | AWS region                                          |
| `bucket`          | `string`  | ✅ Yes   | —                      | Bucket to poll                                      |
| `prefix`          | `string`  | ❌ No    | —                      | Optional key prefix filter                          |
| `intervalMs`      | `number`  | ❌ No    | `15000`                | Poll interval in milliseconds                       |
| `emitOnStart`     | `boolean` | ❌ No    | `false`                | If `false`, first poll warms cache without emitting |
| `pageSize`        | `number`  | ❌ No    | `1000`                 | Max keys per S3 list request                        |

### Trigger Behavior

1. On start, the node restores prior seen keys from workflow context.
2. It lists objects under `bucket/prefix` (with pagination).
3. It filters to keys not in cache.
4. It emits an array of newly seen objects.
5. It persists updated cache and schedules the next poll.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

This is a trigger node and does not accept upstream inputs.

### Outputs

| Output   | Type    | Description                                 |
| -------- | ------- | ------------------------------------------- |
| `output` | `array` | Array of newly discovered S3 object entries |

### Output Example

```json
[
  {
    "Key": "inbox/new-file-1.json",
    "LastModified": "2026-03-11T10:15:30.000Z",
    "Size": 2048
  },
  {
    "Key": "inbox/new-file-2.json",
    "LastModified": "2026-03-11T10:15:32.000Z",
    "Size": 784
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Poll every 15 seconds (default)

```json
{
  "bucket": "my-app-bucket",
  "prefix": "inbox/"
}
```

### Emit existing objects on startup

```json
{
  "bucket": "my-app-bucket",
  "prefix": "inbox/",
  "emitOnStart": true,
  "intervalMs": 30000
}
```

### High-volume listing

```json
{
  "bucket": "my-app-bucket",
  "prefix": "events/",
  "pageSize": 1000,
  "intervalMs": 10000
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### No events on first run

Expected when `emitOnStart` is `false`. Existing objects are cached but not emitted.

#### Duplicate events after workflow reset

If workflow context/cache is cleared, the node rebuilds state and can re-emit existing objects (depending on `emitOnStart`).

#### Slow polling on large prefixes

Increase `intervalMs`, narrow `prefix`, or reduce scanned object volume.

<!-- /SECTION: troubleshooting -->

---
node_id: "aws-dynamodb-trigger"
title: "AWS DynamoDB - New Item Trigger"
description: "Poll a DynamoDB table and emit newly discovered items"
category: "triggers"
subcategory: "aws"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - aws
  - dynamodb
  - polling
  - new-item
related_nodes:
  - aws-dynamodb
---

<!-- SECTION: header -->

# AWS DynamoDB - New Item Trigger

> **Category:** Triggers | **Type:** Trigger Node

Scans a DynamoDB table on an interval and emits items not seen before.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AWS DynamoDB - New Item Trigger** node polls a table with `Scan`, keeps an internal cache of seen item identities, and emits only new items for downstream processing.

### Key Features

- Polling-based trigger for DynamoDB
- Optional filter expressions
- Custom key attributes to define item identity
- Persisted seen-item cache per node instance
- Warmup mode via `emitOnStart`

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter                   | Type       | Required | Default      | Description                                   |
| --------------------------- | ---------- | -------- | ------------ | --------------------------------------------- |
| `region`                    | `string`   | ✅ Yes   | `eu-north-1` | AWS region                                    |
| `accessKeyId`               | `string`   | ❌ No    | —            | AWS access key ID                             |
| `secretAccessKey`           | `string`   | ❌ No    | —            | AWS secret access key                         |
| `tableName`                 | `string`   | ✅ Yes   | —            | Table to scan                                 |
| `keyAttributes`             | `string[]` | ❌ No    | —            | Attributes used to build stable item identity |
| `filterExpression`          | `string`   | ❌ No    | —            | Scan filter expression                        |
| `expressionAttributeValues` | `any`      | ❌ No    | —            | Values for filter expression (JSON/object)    |
| `intervalMs`                | `number`   | ❌ No    | `15000`      | Poll interval in milliseconds                 |
| `emitOnStart`               | `boolean`  | ❌ No    | `false`      | If `false`, first poll only warms cache       |
| `pageSize`                  | `number`   | ❌ No    | `100`        | Number of items per scan page                 |

### Trigger Behavior

1. Restores seen-item cache from workflow context.
2. Scans table pages (`ScanCommand`) with optional filter.
3. Converts DynamoDB items to plain objects.
4. Computes item identity using:
   - `keyAttributes` subset (if provided), or
   - full item JSON.
5. Emits only items not present in cache.
6. Saves updated cache and schedules next poll.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

This is a trigger node and does not accept upstream input.

### Outputs

| Output   | Type    | Description                            |
| -------- | ------- | -------------------------------------- |
| `output` | `array` | Array of newly detected DynamoDB items |

### Output Example

```json
[
  {
    "pk": "ORDER#1002",
    "sk": "META",
    "status": "created",
    "total": 42
  },
  {
    "pk": "ORDER#1003",
    "sk": "META",
    "status": "created",
    "total": 78
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Basic polling trigger

```json
{
  "tableName": "orders",
  "intervalMs": 15000
}
```

### Stable identity from partition/sort keys

```json
{
  "tableName": "orders",
  "keyAttributes": ["pk", "sk"],
  "intervalMs": 10000
}
```

### Emit existing rows on startup

```json
{
  "tableName": "orders",
  "emitOnStart": true,
  "pageSize": 200
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### No output on first start

Expected when `emitOnStart=false`. The first scan populates the cache.

#### Repeated duplicates

Set `keyAttributes` to stable unique keys (for example partition/sort keys). Without this, full item JSON is used as identity.

#### Filter expression not applied

Verify `expressionAttributeValues` is valid DynamoDB AttributeValue JSON expected by the SDK for scan expressions.

<!-- /SECTION: troubleshooting -->

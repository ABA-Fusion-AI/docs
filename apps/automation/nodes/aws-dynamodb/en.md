---
node_id: "aws-dynamodb"
title: "AWS DynamoDB"
description: "Create tables and run item/query operations on AWS DynamoDB"
category: "integrations"
subcategory: "aws"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - aws
  - dynamodb
  - database
  - nosql
related_nodes:
  - aws-dynamodb-trigger
---

<!-- SECTION: header -->

# AWS DynamoDB

> **Category:** Integrations | **Type:** Action Node

Interact with DynamoDB tables using `create`, `put`, `get`, `update`, `delete`, `query`, and `scan`.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **AWS DynamoDB** node provides table creation and data operations for a single configured table.

### Key Features

- Create new tables with on-demand or provisioned throughput
- Put/get/update/delete single items
- Query and scan with pagination
- Automatic marshalling/unmarshalling between JSON and DynamoDB attribute maps

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter                   | Type     | Required | Default           | Description                                                         |
| --------------------------- | -------- | -------- | ----------------- | ------------------------------------------------------------------- |
| `region`                    | `string` | ✅ Yes   | `eu-north-1`      | AWS region                                                          |
| `accessKeyId`               | `string` | ❌ No    | —                 | AWS access key ID                                                   |
| `secretAccessKey`           | `string` | ❌ No    | —                 | AWS secret access key                                               |
| `tableName`                 | `string` | ✅ Yes   | —                 | Target DynamoDB table name                                          |
| `operation`                 | `enum`   | ❌ No    | `get`             | One of: `create`, `put`, `get`, `update`, `delete`, `query`, `scan` |
| `attributeDefinitions`      | `any`    | ✅ Yes\* | —                 | Required for `create`                                               |
| `keySchema`                 | `any`    | ✅ Yes\* | —                 | Required for `create`                                               |
| `billingMode`               | `enum`   | ❌ No    | `PAY_PER_REQUEST` | `PAY_PER_REQUEST` or `PROVISIONED` (`create`)                       |
| `readCapacity`              | `number` | ❌ No    | `5`\*             | Used when `billingMode=PROVISIONED`                                 |
| `writeCapacity`             | `number` | ❌ No    | `5`\*             | Used when `billingMode=PROVISIONED`                                 |
| `tags`                      | `any`    | ❌ No    | —                 | Optional table tags (`create`)                                      |
| `key`                       | `any`    | ✅ Yes\* | —                 | Required for `get`, `update`, `delete`                              |
| `item`                      | `any`    | ✅ Yes\* | —                 | Required for `put`                                                  |
| `updateExpression`          | `string` | ✅ Yes\* | —                 | Required for `update`                                               |
| `expressionAttributeValues` | `any`    | ❌ No    | —                 | For `update`, `query`, `scan`, and `put`                            |
| `conditionExpression`       | `string` | ❌ No    | —                 | For `put`, `update`, `delete`                                       |
| `filterExpression`          | `string` | ❌ No    | —                 | For `query`, `scan`                                                 |
| `limit`                     | `number` | ❌ No    | —                 | For `query`, `scan`                                                 |
| `exclusiveStartKey`         | `any`    | ❌ No    | —                 | For `query`, `scan` pagination                                      |

\* Required only for the operations indicated.

### Operation Behavior

| Operation | Behavior                                                                              | Output                        |
| --------- | ------------------------------------------------------------------------------------- | ----------------------------- |
| `create`  | Creates table from `attributeDefinitions`, `keySchema`, billing config, optional tags | `{ raw }`                     |
| `put`     | Writes one item using `PutItem`                                                       | `{ raw }`                     |
| `get`     | Reads one item by key using `GetItem`                                                 | `{ item }`                    |
| `update`  | Updates item using `UpdateExpression` and returns new attributes                      | `{ attributes }`              |
| `delete`  | Deletes one item by key                                                               | `{ raw }`                     |
| `query`   | Executes `QueryCommand` with optional filters and pagination                          | `{ items, lastEvaluatedKey }` |
| `scan`    | Executes `ScanCommand` with optional filters and pagination                           | `{ items, lastEvaluatedKey }` |

### Notes

- `tableName` is validated against `^[a-zA-Z0-9_.-]+$` (no spaces).
- `attributeDefinitions`, `keySchema`, and `tags` can be passed as objects/arrays or JSON strings.
- For `query`, this implementation uses `key` as a raw `KeyConditionExpression` string.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input   | Type  | Description            |
| ------- | ----- | ---------------------- |
| `input` | `any` | Optional upstream data |

### Outputs

| Output   | Type    | Description                        |
| -------- | ------- | ---------------------------------- |
| `output` | `any`   | Selected operation result          |
| `error`  | `Error` | Emitted on validation/AWS failures |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Put an item

```json
{
  "region": "eu-north-1",
  "tableName": "orders",
  "operation": "put",
  "item": {
    "pk": "ORDER#1001",
    "sk": "META",
    "status": "created",
    "total": 125.5
  }
}
```

### Get an item

```json
{
  "tableName": "orders",
  "operation": "get",
  "key": {
    "pk": "ORDER#1001",
    "sk": "META"
  }
}
```

### Update an item

```json
{
  "tableName": "orders",
  "operation": "update",
  "key": {
    "pk": "ORDER#1001",
    "sk": "META"
  },
  "updateExpression": "SET #s = :s",
  "expressionAttributeValues": {
    ":s": "shipped"
  }
}
```

### Scan with filter

```json
{
  "tableName": "orders",
  "operation": "scan",
  "filterExpression": "#st = :st",
  "expressionAttributeValues": {
    ":st": "created"
  },
  "limit": 50
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Invalid table name

Use only letters, numbers, `_`, `.`, and `-` in `tableName`.

#### Conditional check failed

Your `conditionExpression` evaluated to false. Verify condition values and item state.

#### Query returns validation errors

Ensure `key` (used as `KeyConditionExpression`) is a valid DynamoDB expression string and that matching expression values are provided.

<!-- /SECTION: troubleshooting -->

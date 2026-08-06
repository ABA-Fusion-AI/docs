---
node_id: "cosmos-mongo-manage"
title: "Cosmos MongoDB Manage"
description: "Create or drop databases and collections in Azure Cosmos DB for MongoDB."
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, mongodb, database]
related_nodes:
  - cosmos-mongo-query
---

# Cosmos MongoDB Manage

Creates or drops MongoDB databases and collections through the MongoDB driver. Database, collection, and index names are passed through driver APIs rather than concatenated into query text.

## Configuration

- `connectionString`: Azure Cosmos DB for MongoDB connection string.
- `mode`: `database` or `collection`.
- `databaseManage`: Database name and `ensure` or `drop` action when using database mode.
- `collectionManage`: Database, collection, action, optional collection options, and optional indexes when using collection mode.

The `drop` action is destructive. Use a narrowly scoped database account and keep credentials in the platform secret store.

## Workflow example

See [example.workflow.json](./example.workflow.json) for a minimal workflow containing this node.

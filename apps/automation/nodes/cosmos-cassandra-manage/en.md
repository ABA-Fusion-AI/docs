---
node_id: "cosmos-cassandra-manage"
title: "Cosmos Cassandra Manage"
description: "Safely manage Cassandra keyspaces and tables in Cosmos DB."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, cassandra, security]
---

# Cosmos Cassandra Manage

Keyspace, table, and datacenter identifiers are validated; replication factors and ports must be positive integers. Because table create/alter definitions contain raw CQL fragments, `tableSchema` requires `acknowledgeRisk: true`. Generated CQL is no longer returned in node output.

```fusion-workflow
src: example.workflow.json
```

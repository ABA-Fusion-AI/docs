---
node_id: "cosmos-mongo-query"
title: "Cosmos Mongo Query"
description: "Run protected MongoDB find and aggregate operations against Cosmos DB."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, mongodb, security]
---

# Cosmos Mongo Query

Supports find and aggregate operations. Server-side code operators such as `$where`, `$function`, and `$accumulator` are rejected recursively. Pagination values must be integers. Allowlist user-selectable fields and operators before placing untrusted objects into filters or pipelines.

```fusion-workflow
src: example.workflow.json
```

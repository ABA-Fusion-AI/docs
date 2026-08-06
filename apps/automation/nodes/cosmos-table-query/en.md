---
node_id: "cosmos-table-query"
title: "Cosmos Table Query"
description: "Query Cosmos DB Table API or Azure Tables safely."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, table-api, security]
---

# Cosmos Table Query

Entity lookup mode passes partition and row keys directly to the SDK. Raw OData `filter` strings require `acknowledgeRisk: true`. Selected property names are validated and `top` must be a positive integer. Do not construct filters from untrusted text.

```fusion-workflow
src: example.workflow.json
```

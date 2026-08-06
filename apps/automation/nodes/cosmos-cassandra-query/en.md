---
node_id: "cosmos-cassandra-query"
title: "Cosmos Cassandra Query"
description: "Execute prepared CQL queries against Cosmos DB Cassandra API."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, cassandra, security]
---

# Cosmos Cassandra Query

Custom CQL requires `acknowledgeRisk: true`. Use `?` placeholders and provide positional values in `params`; the node executes the statement as a prepared query. Pagination values must be positive integers. Never concatenate workflow data into CQL.

```fusion-workflow
src: example.workflow.json
```

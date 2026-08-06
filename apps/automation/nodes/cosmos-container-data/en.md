---
node_id: "cosmos-container-data"
title: "Cosmos Container Data"
description: "Read, insert, and safely query items in a Cosmos DB container."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, container, security]
---

# Cosmos Container Data

Supports item lookup, listing, insertion, structured `getBy`, and custom Cosmos SQL queries.

`getBy` safely encodes each property-path segment and binds the comparison value as `@value`. Custom `query` mode requires `acknowledgeRisk: true` and validates all `@parameter` names. Do not concatenate workflow input into query statements.

```fusion-workflow
src: example.workflow.json
title: Safely read Cosmos container data
```

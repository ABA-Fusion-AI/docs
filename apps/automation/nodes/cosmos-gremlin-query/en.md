---
node_id: "cosmos-gremlin-query"
title: "Cosmos Gremlin Query"
description: "Execute Gremlin traversals with validated bindings."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, gremlin, security]
---

# Cosmos Gremlin Query

Custom Gremlin text requires `acknowledgeRisk: true`. Put dynamic values in `parameters`; binding names must follow identifier syntax. Never concatenate workflow values into the traversal text.

```fusion-workflow
src: example.workflow.json
```

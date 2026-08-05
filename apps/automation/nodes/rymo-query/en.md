---
node_id: "rymo-query"
title: "Rymo Query"
description: "Query PostgreSQL with bound filters, safely quoted identifiers, joins, sorting, and pagination."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [postgresql, database, query, security]
---

# Rymo Query

Rymo Query reads PostgreSQL records without requiring generated schemas. Filter and search values are sent to PostgreSQL as bound parameters. Schema, table, field, alias, sorting, grouping, JSON, join, and cursor identifiers are quoted safely.

## Secure configuration

- Define filters as JSON objects, for example `{ "status": { "equals": "active" } }`.
- Supported operators include `equals`, `not`, `in`, `notIn`, `lt`, `lte`, `gt`, `gte`, `contains`, `startsWith`, and `endsWith`.
- JOIN conditions must be object mappings such as `{ "t.roleId": "roles.id" }`. Raw JOIN strings are rejected.
- SELECT fields, grouping fields, and sorting fields must be identifiers. Raw expressions are rejected.
- Sorting directions are limited to `ASC` and `DESC`.
- Raw CTE queries and raw SQL require `acknowledgeRisk: true`.
- Put dynamic raw-query values in `rawSQLParams` and reference them using `$1`, `$2`, and so on. Never interpolate workflow data into `rawSQL`.

```json
{
  "useFilters": true,
  "filters": "{\"status\":{\"equals\":\"[[Trigger.status]]\"}}",
  "useSorting": true,
  "sortFields": "createdAt desc"
}
```

Use a read-only or least-privilege PostgreSQL role whenever possible.

```fusion-workflow
./example.workflow.json
```

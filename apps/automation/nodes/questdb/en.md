---
node_id: "questdb"
title: "QuestDB"
description: "Execute explicitly approved SQL or safely insert typed rows into QuestDB."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [questdb, time-series, sql, database, security]
---

# QuestDB

The QuestDB node connects to the QuestDB REST API and supports raw SQL execution and typed row insertion.

## Insert Rows

Provide a table name, row fields, and an optional column specification such as `symbol:symbol,price:double,ts:timestamp`.

- Table and column identifiers are quoted and escaped.
- Text, symbol, character, date, and timestamp literals escape single quotes.
- Numeric, boolean, and column-type inputs are validated before SQL is generated.
- Empty rows, duplicate column specifications, and unsupported types are rejected.
- Generated SQL and inserted values are not written to logs or returned in node output.

## Execute SQL Query

This operation executes the supplied SQL text exactly as written and therefore requires `executeSqlParams.acknowledgeRisk: true`. Do not concatenate untrusted workflow input into the query. Use a least-privilege QuestDB account and prefer **Insert Rows** for dynamic data.

The optional result limit must be an integer between 1 and 100,000.

```fusion-workflow
./example.workflow.json
```

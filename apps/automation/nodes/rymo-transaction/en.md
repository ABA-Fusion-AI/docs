---
node_id: "rymo-transaction"
title: "Rymo Transaction"
description: "Execute parameterized PostgreSQL statements in a transaction with rollback on failure."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [postgresql, database, transaction, security]
---

# Rymo Transaction

Rymo Transaction executes multiple PostgreSQL statements atomically. It commits only when every statement succeeds and rolls back on failure.

## Secure configuration

Because this node executes raw SQL, `acknowledgeRisk` must be `true`. Prefer query objects containing separate SQL and parameter arrays:

```json
[
  {
    "sql": "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
    "params": [100, "source-account-id"]
  },
  {
    "sql": "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
    "params": [100, "destination-account-id"]
  }
]
```

Never interpolate workflow input into the SQL string. Use `$1`, `$2`, and corresponding `params`. Configure a statement timeout and use a least-privilege database role.

```fusion-workflow
./example.workflow.json
```

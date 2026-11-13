---
node_id: "mongodb-v2"
title: "MongoDB"
description: "Run a MongoDB operation with native query syntax: find, aggregate, insert, update, replace, delete, find-and-modify, bulk write or a multi-document transaction."
category: "data"
subcategory: "databases"
version: "2.0.0"
language: "en"
last_updated: "2026-11-13"
author: "Fusion Team"
tags:
  - mongodb
  - database
  - nosql
  - document
  - aggregation
  - transaction
  - action
related_nodes:
  - mongodb-v2-trigger
  - mongodb-action
  - split-out
  - function
---

<!-- SECTION: header -->
# MongoDB

> **Category:** Data | **Type:** Action Node

Run one MongoDB operation per execution on the official MongoDB driver (7.x), with MongoDB's own query language: filters, update operators, aggregation pipelines, bulk writes and transactions.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **MongoDB** node (v2) replaces [MongoDB Action](/docs/mongodb-action) for new workflows. Values are written as **MongoDB Extended JSON**, so filters, updates and pipelines look exactly like they do in the MongoDB shell, and types such as ObjectId and dates survive the round trip.

### Key Features

- **18 operations:** find, findOne, countDocuments, estimatedDocumentCount, distinct, aggregate, insertOne, insertMany, updateOne, updateMany, replaceOne, deleteOne, deleteMany, findOneAndUpdate, findOneAndReplace, findOneAndDelete, bulkWrite, transaction
- **Native syntax:** every query and update operator (`$gt`, `$in`, `$set`, `$inc`, `$push`, …), update pipelines and array filters
- **Extended JSON types:** `{"$oid": "…"}`, `{"$date": "…"}`, `{"$numberDecimal": "…"}` in, and the same out, so an `_id` from one step can be used as a filter in the next
- **Transactions:** several operations on several collections, all committed or none
- **Shared connection pool:** every MongoDB v2 node with the same URI and settings shares one pool per worker
- **Safety:** updates and deletes need an explicit filter, server-side JavaScript (`$where`, `$function`, `$accumulator`) is refused, and `$out`/`$merge` must be allowed explicitly

### Use Cases

- Query, insert and update documents with real MongoDB operators
- Upsert records from webhooks or other systems
- Run aggregation pipelines for reports
- Move money or stock between documents atomically with a transaction
- Write thousands of mixed changes in one round trip with bulkWrite

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `uri` | `string` | ✅ Yes | — | Connection string, e.g. `mongodb://user:pass@host:27017/?authSource=admin` or `mongodb+srv://…`. Prefer a secret reference to a literal password. |
| `database` | `string` | ✅ Yes | — | Database name. |
| `serverSelectionTimeoutMS` | `number` | ❌ No | `10000` | How long to wait for a usable server before failing. |
| `connectTimeoutMS` | `number` | ❌ No | `10000` | Connection timeout. |
| `socketTimeoutMS` | `number` | ❌ No | `60000` | Fails an operation whose connection stays silent this long, so nothing hangs forever. |
| `maxPoolSize` | `number` | ❌ No | `20` | Connections shared by every MongoDB v2 node with the same URI and settings in a worker. |

### Operation

| Parameter | Type | Shown for | Description |
|-----------|------|-----------|-------------|
| `operation` | `enum` | always | The operation to run (default `find`). |
| `collection` | `string` | all but `transaction` | Collection name. |
| `filter` | Extended JSON | find, findOne, countDocuments, distinct, updates, replace, deletes, findOneAnd* | Query filter, e.g. `{"status": "active", "age": {"$gte": 18}}`. **Required for updates and deletes**; write `{}` to target every document. |
| `projection` | Extended JSON | find, findOne, findOneAnd* | Fields to include or exclude, e.g. `{"name": 1, "_id": 0}`. |
| `sort` | Extended JSON | find, findOne, findOneAnd* | e.g. `{"createdAt": -1}`. |
| `skip` | `number` | find | Documents to skip. |
| `limit` | `number` | find, aggregate | Maximum documents returned (default `1000`). `0` means no limit: the whole result is loaded into the workflow. |
| `field` | `string` | distinct | The field whose distinct values are returned. |
| `pipeline` | Extended JSON | aggregate | Stages, e.g. `[{"$match": {…}}, {"$group": {…}}]`. |
| `allowWriteStages` | `boolean` | aggregate | Allow `$out` and `$merge` (default `false`). |
| `document` | Extended JSON | insertOne | The document to insert. |
| `documents` | Extended JSON | insertMany | Array of documents. |
| `update` | Extended JSON | updateOne, updateMany, findOneAndUpdate | Update operators, e.g. `{"$set": {"status": "done"}, "$inc": {"count": 1}}`, or an update pipeline (array). |
| `replacement` | Extended JSON | replaceOne, findOneAndReplace | The whole new document (no operators). |
| `upsert` | `boolean` | updates, replace, findOneAndUpdate/Replace | Insert when nothing matches (default `false`). |
| `arrayFilters` | Extended JSON | updateOne, updateMany, findOneAndUpdate | Filters for `$[identifier]`, e.g. `[{"elem.grade": {"$gte": 85}}]`. |
| `returnDocument` | `enum` | findOneAndUpdate, findOneAndReplace | `after` (default) or `before` the change. |
| `operations` | Extended JSON | bulkWrite | Mixed writes, e.g. `[{"insertOne": {"document": {…}}}, {"updateOne": {"filter": {…}, "update": {…}}}, {"deleteMany": {"filter": {…}}}]`. |
| `ordered` | `boolean` | bulkWrite, insertMany | Stop at the first error (default `true`) or continue with the rest. |
| `steps` | Extended JSON | transaction | Operations run atomically, each with its own `collection` and `operation`. |
| `maxTimeMS` | `number` | reads, aggregate, findOneAnd*, transaction | Server-side time limit (default `30000`). `0` disables it. |
| `options` | Extended JSON | always | Extra driver options, e.g. `{"collation": {"locale": "fr"}, "hint": {"email": 1}, "comment": "nightly job"}`. |

### Extended JSON in short

| Type | Write |
|------|-------|
| ObjectId | `{"$oid": "65f1c0ffee0000000000abcd"}` |
| Date | `{"$date": "2026-01-01T00:00:00Z"}` |
| Decimal | `{"$numberDecimal": "19.99"}` |
| Long | `{"$numberLong": "9007199254740993"}` |

A field may also be an expression that resolves to an object, e.g. `{{ outputs.Build.success }}`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Runs the operation once per incoming event. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The operation's result, as relaxed Extended JSON. |
| `error` | `object` | `{ name, message, code }`, e.g. `MongoServerError` with code `11000` for a duplicate key. |

### Results by operation

| Operation | Result |
|-----------|--------|
| find, aggregate | Array of documents |
| findOne, findOneAnd* | The document, or `null` |
| countDocuments, estimatedDocumentCount | `{ "count": n }` |
| distinct | `{ "values": [...] }` |
| insertOne | `{ "acknowledged": true, "insertedId": {"$oid": "…"} }` |
| insertMany | `{ "acknowledged": true, "insertedCount": n, "insertedIds": { "0": {"$oid": "…"}, … } }` |
| updates, replaceOne | `{ "matchedCount", "modifiedCount", "upsertedCount", "upsertedId" }` |
| deletes | `{ "deletedCount": n }` |
| bulkWrite | `{ "insertedCount", "matchedCount", "modifiedCount", "deletedCount", "upsertedCount", "insertedIds", "upsertedIds" }` |
| transaction | `{ "committed": true, "results": [ one result per step ] }` |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Upsert a customer

```json
{
  "operation": "updateOne",
  "collection": "customers",
  "filter": "{\"email\": \"alice@example.com\"}",
  "update": "{\"$set\": {\"name\": \"Alice\"}, \"$inc\": {\"visits\": 1}}",
  "upsert": true
}
```

### Revenue per day

```json
{
  "operation": "aggregate",
  "collection": "orders",
  "pipeline": "[{\"$match\": {\"status\": \"paid\"}}, {\"$group\": {\"_id\": {\"$dateToString\": {\"format\": \"%Y-%m-%d\", \"date\": \"$createdAt\"}}, \"revenue\": {\"$sum\": \"$total\"}}}, {\"$sort\": {\"_id\": 1}}]"
}
```

### Transfer between two accounts

```json
{
  "operation": "transaction",
  "steps": "[{\"collection\": \"accounts\", \"operation\": \"updateOne\", \"filter\": {\"_id\": 1}, \"update\": {\"$inc\": {\"balance\": -50}}}, {\"collection\": \"accounts\", \"operation\": \"updateOne\", \"filter\": {\"_id\": 2}, \"update\": {\"$inc\": {\"balance\": 50}}}, {\"collection\": \"ledger\", \"operation\": \"insertOne\", \"document\": {\"from\": 1, \"to\": 2, \"amount\": 50}}]"
}
```

If any step fails, none of them is applied. Transactions need a replica set or sharded cluster.

### Field names that contain a dot

MongoDB reads `{"a.b": 1}` in a filter or `$set` as the path `a` → `b`. A document can still store a key literally named `"a.b"` (insert and replace keep it as is). To read or change such a key, use `$getField` and `$setField`:

```json
{ "filter": "{\"$expr\": {\"$eq\": [{\"$getField\": \"sensor.temp\"}, 21.5]}}" }
```

```json
{ "update": "[{\"$replaceWith\": {\"$setField\": {\"field\": \"sensor.temp\", \"input\": \"$$ROOT\", \"value\": 22}}}]" }
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Upsert a customer, then read the active ones
```

**How it flows:**
1. **Upsert Customer** creates or updates Alice's document, counting her visits.
2. **Active Customers** returns up to 50 active customers, most visits first.
3. Errors from either step go to the error log.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

- Store the connection string, including its password, as a secret.
- Give the database user only the roles the workflow needs (for example `readWrite` on one database).
- `$where`, `$function` and `$accumulator` (server-side JavaScript) are always refused.
- `$out` and `$merge` run only with **Allow Write Stages** enabled.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

#### `"filter" is required … write {} to target every document`

**Cause:** An update or delete has no filter. This is deliberate, so an empty field never changes a whole collection.

**Solution:** Write the filter, or `{}` to target every document.

#### `MongoServerError` code `11000`

**Cause:** Duplicate key on a unique index.

**Solution:** Use `upsert`, or handle the error output.

#### `MongoServerError` code `50` (MaxTimeMSExpired)

**Cause:** The operation took longer than **Max Time**.

**Solution:** Add an index, narrow the filter, or raise **Max Time**.

#### `BSONSizeError: A document or command is larger than MongoDB's 16 MB BSON limit`

**Cause:** A document (or an insert batch command) is over 16 MB.

**Solution:** Store large content outside the document (for example GridFS or object storage), or split it.

#### `MongoServerSelectionError`

**Cause:** No server reachable within **Server Selection Timeout**: wrong host, network, or a database that is down.

**Solution:** Check the URI and that the database is reachable from the workers.

#### Only 1000 documents returned

**Cause:** The default **Limit** for find and aggregate.

**Solution:** Set **Limit** to what you need, or `0` for no limit (the whole result is held in memory).

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [MongoDB Trigger](/docs/mongodb-v2-trigger) – Start a workflow when documents change
- [MongoDB Action](/docs/mongodb-action) – The previous MongoDB node
- [Split Out](/docs/split-out) – Process each document of a result separately

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2026-11-13 | New node on the official MongoDB driver 7.x: Extended JSON, 18 operations, bulk write, transactions, shared connection pool |

<!-- /SECTION: changelog -->

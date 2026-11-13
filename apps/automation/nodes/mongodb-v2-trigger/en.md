---
node_id: "mongodb-v2-trigger"
title: "MongoDB Trigger"
description: "Start a workflow when documents change in a MongoDB collection or database: inserts, updates, replaces and deletes, from a change stream."
category: "triggers"
subcategory: "database-events"
version: "2.0.0"
language: "en"
last_updated: "2026-11-13"
author: "Fusion Team"
tags:
  - mongodb
  - trigger
  - change-stream
  - database
  - events
  - nosql
related_nodes:
  - mongodb-v2
  - function
  - split-out
---

<!-- SECTION: header -->
# MongoDB Trigger

> **Category:** Triggers | **Type:** Trigger Node

Emit one event for every change in a MongoDB collection, or in a whole database, using a MongoDB change stream.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **MongoDB Trigger** watches a collection (or every collection of a database) and starts the workflow for each insert, update, replace or delete. It runs on the official MongoDB driver (7.x).

### Key Features

- **No missed changes on reconnect:** the trigger remembers the last change it emitted (its resume token). After a network error or a database restart it reopens from there, so no change is lost and none is emitted twice.
- **Pause and resume:** pausing the workflow stops events; resuming delivers the changes made while paused.
- **Server-side filtering:** add change-stream stages, so only the changes you want leave the database.
- **Full documents:** for updates, get the whole current document, not only the changed fields.

### Requirements

Change streams need a **replica set** or **sharded cluster**. A single-node replica set is enough. A standalone server does not support them.

### Use Cases

- React to new orders, sign-ups or tickets the moment they are written
- Sync changes from MongoDB to another system
- Audit trail of every change in a collection
- Notify a team when a document reaches a status

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `uri` | `string` | ✅ Yes | — | Connection string of a replica set or sharded cluster. Prefer a secret reference. |
| `database` | `string` | ✅ Yes | — | Database to watch. |
| `collection` | `string` | ❌ No | — | Collection to watch. Leave empty to watch every collection of the database. |
| `operationTypes` | `array` | ❌ No | `insert, update, replace, delete` | Change types to emit. Also available: `drop`, `rename`, `dropDatabase`, `invalidate`. |
| `pipeline` | Extended JSON | ❌ No | — | Extra change-stream stages, e.g. `[{"$match": {"fullDocument.status": "urgent"}}]`. |
| `fullDocument` | `enum` | ❌ No | `updateLookup` | For updates: `updateLookup` includes the current whole document; `default` sends only the changed fields; `whenAvailable` / `required` use stored post-images. |
| `fullDocumentBeforeChange` | `enum` | ❌ No | `off` | Include the document as it was before the change (`whenAvailable` or `required`). Needs `changeStreamPreAndPostImages` enabled on the collection. |
| `serverSelectionTimeoutMS`, `connectTimeoutMS`, `socketTimeoutMS`, `maxPoolSize` | `number` | ❌ No | as [MongoDB](/docs/mongodb-v2) | Connection settings. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | One event per change (relaxed Extended JSON). |
| `error` | `object` | `{ name, message, code }` when the stream fails, for example while the database restarts. The trigger then reopens by itself, from where it stopped. |

### Example event (insert)

```json
{
  "operationType": "insert",
  "ns": { "db": "shop", "coll": "orders" },
  "documentKey": { "_id": { "$oid": "6af72f3b6ed1519cb329f23b" } },
  "fullDocument": { "_id": { "$oid": "6af72f3b6ed1519cb329f23b" }, "total": 129.9, "status": "new" },
  "clusterTime": { "$timestamp": { "t": 1794584268, "i": 1 } },
  "resumeToken": { "_data": "8269…" }
}
```

An update also carries `updateDescription` (`updatedFields`, `removedFields`), and `fullDocument` when **Full Document** is `updateLookup`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: React to new and changed orders
```

**How it flows:**
1. **Watch Orders** emits inserts, updates and replaces in `orders` whose total is 100 or more (filtered in the database by the pipeline).
2. **Summary** keeps the order id, the change type and the total.
3. Stream errors go to the error log while the trigger reconnects.

### Common Patterns

- **Filter in the pipeline, not after:** a `$match` stage keeps unwanted changes inside the database.
- **Wire the error output** to a log or alert: it reports outages such as a database restart (code `11600`), after which the trigger continues by itself.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

- Store the connection string as a secret.
- The database user needs the `changeStream` and `find` privileges on what it watches (the built-in `read` role is enough).

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

#### `The $changeStream stage is only supported on replica sets`

**Cause:** The server is a standalone instance.

**Solution:** Use a replica set or sharded cluster. A single-node replica set works.

#### Error code `11600` (interrupted at shutdown)

**Cause:** The database restarted or stepped down.

**Solution:** None needed: the trigger reopens from its last position. Changes made meanwhile are delivered.

#### Error code `286` (ChangeStreamHistoryLost)

**Cause:** The trigger was disconnected for so long that its position fell out of the database's history (oplog).

**Solution:** The trigger continues from the current moment; changes in the gap cannot be recovered. Size the oplog for the longest outage you expect.

#### Updates arrive without the whole document

**Cause:** **Full Document** is `default`.

**Solution:** Use `updateLookup`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [MongoDB](/docs/mongodb-v2) – Read and write documents
- [Function](/docs/function) – Shape each change event
- [Split Out](/docs/split-out) – Split a list from a changed document into items

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2026-11-13 | New trigger on the official MongoDB driver 7.x with resume after reconnect and pause/resume |

<!-- /SECTION: changelog -->

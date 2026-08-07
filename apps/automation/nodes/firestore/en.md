---
node_id: "firestore"
title: "Firestore"
description: "Perform Firestore document and collection operations including set, get, update, delete, and query."
category: "Firebase"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - firestore
  - firebase
  - database
  - nosql
  - document
  - query
  - cloud
  - action
related_nodes:
  - function
  - if
  - http-request
---

<!-- SECTION: overview -->
# Firestore

> **Category:** database-nodes | **Type:** Action Node

Perform Firestore document and collection operations using **Google Firebase Firestore**.

The **Firestore** node allows workflows to create, retrieve, update, delete, and query Firestore documents. It supports common Firestore operations while automatically initializing a Firebase application using the provided project credentials.

### Supported Operations

- Set Document
- Get Document
- Update Document
- Delete Document
- Query Documents

### Use Cases

- Store workflow results
- Retrieve user profiles
- Update application data
- Delete expired records
- Query documents with filters
- Build backend automation using Firebase

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Firebase API key. |
| `authDomain` | `string` | ❌ No | — | Firebase Authentication domain. |
| `projectId` | `string` | ✅ Yes | — | Firebase project ID. |
| `storageBucket` | `string` | ❌ No | — | Firebase Storage bucket. |
| `messagingSenderId` | `string` | ❌ No | — | Firebase Messaging sender ID. |
| `appId` | `string` | ❌ No | — | Firebase application ID. |
| `operation` | `string` | ✅ Yes | — | Firestore operation to perform. |
| `collectionId` | `string` | ✅ Yes | — | Firestore collection name. |
| `documentId` | `string` | Required for document operations | — | Document identifier. |
| `data` | `string` | Required for set/update | — | JSON string representing the document data. |
| `query` | `object` | Required for query operation | — | Firestore query configuration. |

### Supported Operations

| Operation | Description |
|----------|-------------|
| `set document` | Creates or replaces a document. |
| `get document` | Retrieves a document by ID. |
| `update document` | Updates existing document fields. |
| `delete document` | Deletes a document. |
| `query documents` | Returns documents matching query constraints. |

### Query Object

The `query` parameter supports the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `where` | `array` | List of filter conditions. |
| `orderBy` | `array` | Sort definitions. |
| `limit` | `number` | Maximum number of returned documents. |

#### Where Clause

Each where condition supports:

| Field | Description |
|-------|-------------|
| `field` | Document field name |
| `operator` | Firestore comparison operator |
| `value` | Comparison value |
| `arrayValue` | Used by array operators (`in`, `not-in`, `array-contains-any`) |

Supported operators include:

- `==`
- `!=`
- `<`
- `<=`
- `>`
- `>=`
- `in`
- `not-in`
- `array-contains`
- `array-contains-any`

#### Order By

| Field | Description |
|------|-------------|
| `field` | Field to sort by |
| `direction` | `asc` or `desc` |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This node does not require workflow input. All required values are provided through node configuration.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Successful operation result. |
| `error` | `Error` | Returned if validation or Firestore operations fail. |

### Output Examples

#### Set Document

**Configuration**

```json
{
  "operation": "set document",
  "collectionId": "users",
  "documentId": "john",
  "data": "{\"name\":\"John\",\"age\":30}"
}
```

**Output**

```json
{
  "success": true
}
```

---

#### Get Document

**Configuration**

```json
{
  "operation": "get document",
  "collectionId": "users",
  "documentId": "john"
}
```

**Output**

```json
{
  "name": "John",
  "age": 30
}
```

---

#### Update Document

**Configuration**

```json
{
  "operation": "update document",
  "collectionId": "users",
  "documentId": "john",
  "data": "{\"age\":31}"
}
```

**Output**

```json
{
  "success": true
}
```

---

#### Delete Document

**Configuration**

```json
{
  "operation": "delete document",
  "collectionId": "users",
  "documentId": "john"
}
```

**Output**

```json
{
  "success": true
}
```

---

#### Query Documents

**Configuration**

```json
{
  "operation": "query documents",
  "collectionId": "users",
  "query": {
    "where": [
      {
        "field": "age",
        "operator": ">=",
        "value": "18"
      }
    ],
    "orderBy": [
      {
        "field": "age",
        "direction": "desc"
      }
    ],
    "limit": 5
  }
}
```

**Output**

```json
[
  {
    "id": "john",
    "name": "John",
    "age": 31
  },
  {
    "id": "alice",
    "name": "Alice",
    "age": 28
  }
]
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Store Webhook Data

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "save-user",
      "type": "firestore",
      "config": {
        "operation": "set document",
        "collectionId": "users",
        "documentId": "{{input.id}}",
        "data": "{{input.body}}"
      }
    }
  ]
}
```

### Common Patterns

- Webhook → Store Document
- HTTP Request → Update Firestore
- Query Collection → Process Results
- Retrieve User → Continue Workflow
- Scheduled Job → Delete Old Records
- Form Submission → Save to Firestore

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Collection ID is required

**Cause**

No collection name was configured.

**Solution**

Specify a valid `collectionId`.

---

### Document ID is required

**Cause**

A document operation was selected without providing a document ID.

**Solution**

Provide a `documentId` when using:

- Set Document
- Get Document
- Update Document
- Delete Document

---

### Document data is required

**Cause**

The selected operation requires document data but none was supplied.

**Solution**

Provide a valid JSON string in the `data` parameter.

Example:

```json
{
  "name": "John",
  "age": 30
}
```

---

### Query object is required

**Cause**

The Query Documents operation was selected without a query configuration.

**Solution**

Provide a valid `query` object containing filters, sorting, or limits.

---

### Invalid JSON

**Cause**

The document data contains malformed JSON.

**Solution**

Validate the JSON before passing it to the node.

---

### No such document

**Cause**

The requested document does not exist.

**Solution**

Verify the collection name and document ID before performing the lookup.

---

### Firebase Initialization Failed

**Cause**

Firebase could not be initialized using the provided credentials.

**Possible reasons**

- Invalid API key
- Incorrect project ID
- Missing Firebase configuration
- Network connectivity issues

**Solution**

Verify your Firebase project configuration and credentials.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](./function.md) – Transform Firestore data
- [If](./if.md) – Route workflows based on query results
- [HTTP Request](./http-request.md) – Integrate with external services

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial release |

<!-- /SECTION: changelog -->
---
node_id: "raindrop"
title: "Raindrop.io"
description: "Manage Raindrop.io bookmarks and collections from a workflow"
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - integration
  - raindrop
  - bookmarks
  - collections
  - peer-only
related_nodes:
  - http-request
  - log
---

<!-- SECTION: header -->

# Raindrop.io

> **Category:** Peer-only Integrations | **Type:** Action Node

Manage bookmarks and collections in Raindrop.io from an automation workflow.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **Raindrop.io** node connects to Raindrop.io using an access token. It can read and modify bookmarks and collections, and it exposes the remote-service response through the workflow outputs.

### Key Features

- **Bookmark management:** List, create, retrieve, update, and delete bookmarks
- **Collection management:** List and create collections
- **Token authentication:** Authenticate requests with a Raindrop.io access token
- **Bookmark metadata:** Set a bookmark URL, title, tags, and collection
- **Workflow-ready outputs:** Route successful responses or errors to downstream nodes

### Supported Operations

| Operation | Description |
|-----------|-------------|
| `listBookmarks` | List bookmarks available to the authenticated account |
| `createBookmark` | Create a bookmark from a URL and optional metadata |
| `getBookmark` | Retrieve a bookmark by ID |
| `updateBookmark` | Update a bookmark by ID |
| `deleteBookmark` | Delete a bookmark by ID |
| `listCollections` | List collections available to the authenticated account |
| `createCollection` | Create a collection with a title |

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | — | Raindrop.io operation to execute |
| `accessToken` | `string` | ✅ Yes | — | Raindrop.io API access token |
| `link` | `url` | Operation-dependent | — | Bookmark URL used when creating or updating a bookmark |
| `bookmarkTitle` | `string` | No | — | Bookmark title used when creating or updating a bookmark |
| `tags` | `string` | No | — | Comma-separated bookmark tags |
| `collectionId` | `string` or `number` | No | — | Collection ID used when creating a bookmark |
| `bookmarkId` | `string` or `number` | Operation-dependent | — | Bookmark ID used by `getBookmark`, `updateBookmark`, and `deleteBookmark` |
| `limit` | `number` | No | — | Maximum number of items requested when listing bookmarks |
| `collectionTitle` | `string` | Yes for `createCollection` | — | Name of the collection to create |

### Operation Requirements

| Operation | Parameters commonly used |
|-----------|--------------------------|
| `listBookmarks` | `accessToken`, optional `limit` |
| `createBookmark` | `accessToken`, `link`; optional `bookmarkTitle`, `tags`, `collectionId` |
| `getBookmark` | `accessToken`, `bookmarkId` |
| `updateBookmark` | `accessToken`, `bookmarkId`; update fields such as `link`, `bookmarkTitle`, and `tags` |
| `deleteBookmark` | `accessToken`, `bookmarkId` |
| `listCollections` | `accessToken` |
| `createCollection` | `accessToken`, `collectionTitle` |

> The exact required fields can depend on the selected operation. Configure the parameters relevant to that operation before executing the node.

### Authentication

The node authenticates requests with `accessToken`. Store the token in Fusion's credential system or another protected secret configuration. Do not commit a real token to a workflow file or documentation.

Example placeholder configuration:

```json
{
  "operation": "listBookmarks",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>"
}
```

### Bookmark Parameters

For `createBookmark` and `updateBookmark`, the example workflow uses the following fields:

```json
{
  "link": "https://developer.mozilla.org/",
  "bookmarkTitle": "Documentation MDN",
  "tags": "documentation,web",
  "collectionId": "-1"
}
```

The `tags` value is entered as a comma-separated string. `collectionId` identifies the collection to use when creating a bookmark.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node |

The node's operation and parameters are configured in the node settings. Incoming data can be used as part of a larger workflow, but the example workflow configures each Raindrop.io action explicitly.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Successful Raindrop.io response |
| `error` | `object` or `string` | Validation, authentication, network, or remote-service failure |

The response shape depends on the selected Raindrop.io operation. Bookmark-list and collection-list operations generally return collections of resources, while create, retrieve, update, and delete operations return the corresponding service response.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### List Bookmarks

Retrieve bookmarks from the authenticated Raindrop.io account.

```json
{
  "operation": "listBookmarks",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>"
}
```

### Create a Bookmark

Create a bookmark with a title, tags, and collection.

```json
{
  "operation": "createBookmark",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>",
  "link": "https://developer.mozilla.org/",
  "bookmarkTitle": "Documentation MDN",
  "tags": "documentation,web",
  "collectionId": "-1"
}
```

### Get a Bookmark

Retrieve one bookmark by its ID.

```json
{
  "operation": "getBookmark",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>",
  "bookmarkId": "1848640574"
}
```

### Update a Bookmark

Update the URL, title, or tags of an existing bookmark.

```json
{
  "operation": "updateBookmark",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>",
  "bookmarkId": "1848640574",
  "link": "https://www.wikipedia.org/",
  "bookmarkTitle": "Wikipedia",
  "tags": "reference,web"
}
```

### Delete a Bookmark

Delete an existing bookmark by its ID.

```json
{
  "operation": "deleteBookmark",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>",
  "bookmarkId": "1848640574"
}
```

### List Collections

Retrieve collections from the authenticated account.

```json
{
  "operation": "listCollections",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>"
}
```

### Create a Collection

Create a new collection with a title.

```json
{
  "operation": "createCollection",
  "accessToken": "<RAINDROP_ACCESS_TOKEN>",
  "collectionTitle": "Fusion Documentation"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow contains separate manual-trigger flows for each supported operation. Each Raindrop.io node sends its successful response to a Log node for inspection.

```fusion-workflow
src: example.workflow.json
title: Manage Raindrop.io bookmarks and collections
```

### Common Patterns

- **Read bookmarks:** Manual Trigger → Raindrop.io (`listBookmarks`) → Log
- **Create a bookmark:** Manual Trigger → Raindrop.io (`createBookmark`) → Log
- **Update a bookmark:** Manual Trigger → Raindrop.io (`updateBookmark`) → Log
- **Organize collections:** Manual Trigger → Raindrop.io (`listCollections` or `createCollection`) → Log
- **Handle failures:** Raindrop.io `error` output → Error-handling or notification node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Node is not registered

**Cause:** The package containing the `raindrop` node is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

#### Authentication failed

**Cause:** The access token is missing, expired, malformed, or does not have sufficient permissions.

**Solution:** Configure a valid Raindrop.io access token and verify that it has access to the requested resource.

#### Bookmark ID is invalid

**Cause:** `getBookmark`, `updateBookmark`, or `deleteBookmark` received a missing or invalid `bookmarkId`.

**Solution:** Use the ID returned by Raindrop.io for the target bookmark and remove leading or trailing whitespace.

#### Bookmark creation failed

**Cause:** The `link` is missing or is not a valid URL, or one of the optional values is invalid.

**Solution:** Provide a valid URL and check the title, tags, and collection ID values.

#### Collection creation failed

**Cause:** `collectionTitle` is missing or the account cannot create collections.

**Solution:** Provide a non-empty title and verify the token permissions.

#### Request failed

**Cause:** Raindrop.io is unavailable, the request was rejected, or the network connection failed.

**Solution:** Check the token, operation-specific parameters, service availability, and workflow connectivity.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication error | Missing or invalid access token | Configure a valid protected token |
| Validation error | Missing or malformed operation parameter | Check the selected operation and required fields |
| Bookmark not found | Invalid or inaccessible `bookmarkId` | Use a valid bookmark ID |
| Collection error | Invalid collection data or insufficient permissions | Check `collectionTitle`, `collectionId`, and permissions |
| Network or service error | Request could not reach or be completed by Raindrop.io | Check connectivity and retry later |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [HTTP Request](./http-request.md) - Send requests to APIs that do not have a dedicated node
- [Log](./log.md) - Inspect Raindrop.io responses during workflow development

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Expanded documentation and documented the example workflow operations |

<!-- /SECTION: changelog -->


---
node_id: "zotero"
title: "Zotero Library"
description: "Search and retrieve items and collections from a Zotero User or Group library."
category: "Research / Reference Management"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:

- zotero
- reference-management
- bibliography
- research
- citations
- library
- api

related_nodes:
- manual-trigger
- function
- if
- log
- http-request

---

# Zotero Library

> **Category:** research-nodes | **Type:** Action Node

Search and retrieve bibliographic records and collections from a **Zotero** User or Group library through the Zotero Web API v3.

The **Zotero Library** node supports three operations: `search`, `getItem`, and `collections`. It can search a library by title, creator, year, or full item content; retrieve one item using its Zotero Item Key; or list the library's collections.

### Supported Features

- Access Zotero User and Group libraries
- Search by quick-search text
- Search titles, creators, and years, or search all indexed fields
- Filter results by item type and tag
- Retrieve a specific item using its 8-character Item Key
- List up to 100 collections
- Return simplified, workflow-friendly item objects
- Accept a search query or Item Key from an incoming workflow value
- Use public libraries without an API key when Zotero permissions allow it
- Authenticate access to private libraries with a Zotero Web API key
- Apply request spacing and retry after Zotero `429` rate-limit responses

### Use Cases

- Search academic references stored in Zotero
- Retrieve article metadata, authors, DOI, abstract, and tags
- Build a bibliography or reading-list workflow
- Find all books, theses, journal articles, or conference papers matching a topic
- Route references by tag or item type
- Synchronize Zotero collections with another application or database
- Feed Zotero records into summarization, RAG, citation, or reporting workflows

---

## Configuration

### Base Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | ❌ No | — | Zotero Web API key. Optional for public libraries, but normally required for private libraries. |
| `libraryId` | `string` | ✅ Yes | — | Numeric Zotero User ID or Group ID. This is not a username or email address. |
| `libraryType` | `enum` | ❌ No | `"user"` | Library owner type: `user` or `group`. |
| `operation` | `enum` | ❌ No | `"search"` | Operation to execute: `search`, `getItem`, or `collections`. |

### Search Parameters

These parameters are displayed when `operation` is `search`.

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `query` | `string` | Conditional | — | Quick-search term. If omitted, the node derives it from the incoming workflow value. |
| `searchMode` | `enum` | ❌ No | `"titleCreatorYear"` | `titleCreatorYear` searches titles, creators, and years; `everything` performs a broader full search. |
| `itemType` | `enum` | ❌ No | `"all"` | Filter by `journalArticle`, `book`, `thesis`, `conferencePaper`, or return `all`. |
| `tag` | `string` | ❌ No | — | Return only items containing the specified Zotero tag. |
| `limit` | `number` | ❌ No | `25` | Maximum number of items requested from Zotero. |

The search query is required at runtime, either through `query` or through the incoming data.

### Get Item Parameters

This parameter is displayed when `operation` is `getItem`.

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `itemKey` | `string` | Conditional | — | Zotero Item Key, normally an 8-character value such as `AB12CD34`. If omitted, the node attempts to extract it from incoming data. |

No operation-specific parameter is required for `collections`.

---

## Authentication and Library IDs

### Zotero API Key

Create a Zotero Web API key from your Zotero account settings:

1. Sign in to Zotero.
2. Open **Settings → Feeds/API**.
3. Under **Applications**, create a new private key.
4. Give the key the library permissions required by the workflow.
5. Copy the generated key into `apiKey` or store it in a secret.

The node sends the key using this header:

```text
Authorization: Bearer <apiKey>
```

It also sends `Zotero-API-Version: 3` on every request.

### User Library ID

For a User library, `libraryId` must be the numeric Zotero User ID. It can be found on the Zotero API settings page. Select `user` as `libraryType`.

The resulting API prefix is:

```text
users/<libraryId>
```

### Group Library ID

For a Group library, use the numeric Group ID and select `group` as `libraryType`. The Group ID is visible in the group's URL and Zotero group information.

The resulting API prefix is:

```text
groups/<libraryId>
```

---

## Operations

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| `search` | `/{libraryType}s/{libraryId}/items` | `GET` | Search and filter library items, sorted by date descending. |
| `getItem` | `/{libraryType}s/{libraryId}/items/{itemKey}` | `GET` | Retrieve one item by its Zotero Item Key. |
| `collections` | `/{libraryType}s/{libraryId}/collections` | `GET` | List up to 100 collections from the selected library. |

The base URL for all requests is:

```text
https://api.zotero.org
```

---

## Request Construction

### Search

The node sends these query parameters:

| API Parameter | Source | Behavior |
| ------------- | ------ | -------- |
| `q` | `query` or incoming data | Quick-search value. |
| `qmode` | `searchMode` | Defaults to `titleCreatorYear`. |
| `limit` | `limit` | Defaults to `25`. |
| `sort` | Fixed value | Always `date`. |
| `direction` | Fixed value | Always `desc`. |
| `itemType` | `itemType` | Included only when the value is not `all`. |
| `tag` | `tag` | Included only when a non-empty tag is provided. |

Example request:

```text
GET https://api.zotero.org/users/123456/items?q=machine%20learning&qmode=titleCreatorYear&limit=25&sort=date&direction=desc&itemType=journalArticle
```

### Get Item

The node first uses the configured `itemKey`. If it is empty, it accepts:

- A string input containing the Item Key
- An object with a `key` property
- An object with an `itemKey` property
- An object with an `id` property

### Collections

The node requests the collections endpoint with a fixed `limit=100` query parameter.

---

## Inputs & Outputs

### Inputs

The behavior of incoming workflow data depends on the selected operation.

| Operation | Accepted Input | Behavior |
| --------- | -------------- | -------- |
| `search` | `string` or any value | Used as the search query only when the configured `query` is empty. Non-string data is converted with `String(data)`. |
| `getItem` | `string` | Treated as the Item Key when `itemKey` is empty. |
| `getItem` | `object` | Reads `key`, then `itemKey`, then `id` when `itemKey` is empty. |
| `collections` | Any | Incoming data is ignored. |

### Search Output

```json
{
  "results": [],
  "count": 0,
  "libraryId": "123456"
}
```

| Field | Type | Description |
| ----- | ---- | ----------- |
| `results` | `array` | Formatted Zotero items returned by the search. |
| `count` | `number` | Number of results in this response. |
| `libraryId` | `string` | Library ID used for the request. |

Each formatted item can contain:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `key` | `string` | Zotero Item Key. |
| `itemType` | `string` | Zotero item type. |
| `title` | `string` | Item title, or `Untitled` when no title exists. |
| `authors` | `string[]` | Creator names. |
| `date` | `string` | Publication date stored in Zotero. |
| `publication` | `string` | Publication title, proceedings title, or university. |
| `doi` | `string` | DOI when available. |
| `url` | `string` | Item URL when available. |
| `tags` | `string[]` | Zotero tags. |
| `abstract` | `string` | Abstract note when available. |
| `collections` | `string[]` | Collection keys assigned to the item. |
| `metadata` | `object` | Date added, date modified, version, and parent item information. |

### Get Item Output

Returns one formatted item using the same item structure as a search result.

### Collections Output

Returns an array of simplified collection objects:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `key` | `string` | Zotero Collection Key. |
| `name` | `string` | Collection name. |
| `parentCollection` | `string` or `false` | Parent collection key, or `false` for a top-level collection. |
| `numItems` | `number` | Number of items reported by Zotero for the collection. |

---

## Output Examples

### `search`

```json
{
  "results": [
    {
      "key": "AB12CD34",
      "itemType": "journalArticle",
      "title": "Artificial Intelligence in Education",
      "authors": ["Jane Smith", "Omar Karim"],
      "date": "2025",
      "publication": "Journal of Learning Technologies",
      "doi": "10.1234/example.2025.001",
      "url": "https://example.org/article",
      "tags": ["AI", "education"],
      "abstract": "A study of artificial intelligence in education.",
      "collections": ["ZX90YU12"],
      "metadata": {
        "dateAdded": "2026-08-20T10:00:00Z",
        "dateModified": "2026-08-25T14:30:00Z",
        "version": 42,
        "parentItem": false
      }
    }
  ],
  "count": 1,
  "libraryId": "123456"
}
```

### `getItem`

```json
{
  "key": "AB12CD34",
  "itemType": "book",
  "title": "Research Methods",
  "authors": ["Jane Smith"],
  "date": "2024",
  "publication": "",
  "doi": "",
  "url": "https://example.org/book",
  "tags": ["methodology"],
  "abstract": "",
  "collections": [],
  "metadata": {
    "dateAdded": "2026-08-01T09:00:00Z",
    "dateModified": "2026-08-01T09:00:00Z",
    "version": 3,
    "parentItem": false
  }
}
```

### `collections`

```json
[
  {
    "key": "ZX90YU12",
    "name": "Artificial Intelligence",
    "parentCollection": false,
    "numItems": 18
  },
  {
    "key": "MN34PQ56",
    "name": "Education",
    "parentCollection": "ZX90YU12",
    "numItems": 7
  }
]
```

---

## Configuration Examples

### Search a User Library

```json
{
  "apiKey": "your-zotero-api-key",
  "libraryId": "123456",
  "libraryType": "user",
  "operation": "search",
  "query": "artificial intelligence",
  "searchMode": "titleCreatorYear",
  "itemType": "all",
  "limit": 25
}
```

### Search Journal Articles by Tag

```json
{
  "apiKey": "your-zotero-api-key",
  "libraryId": "123456",
  "libraryType": "user",
  "operation": "search",
  "query": "machine learning",
  "searchMode": "everything",
  "itemType": "journalArticle",
  "tag": "reviewed",
  "limit": 10
}
```

### Search a Public Group Library Without an API Key

```json
{
  "libraryId": "987654",
  "libraryType": "group",
  "operation": "search",
  "query": "climate change",
  "searchMode": "titleCreatorYear",
  "itemType": "all",
  "limit": 25
}
```

This works only when the selected Zotero Group library is publicly readable.

### Retrieve an Item

```json
{
  "apiKey": "your-zotero-api-key",
  "libraryId": "123456",
  "libraryType": "user",
  "operation": "getItem",
  "itemKey": "AB12CD34"
}
```

### List Collections

```json
{
  "apiKey": "your-zotero-api-key",
  "libraryId": "123456",
  "libraryType": "user",
  "operation": "collections"
}
```

---

## Testing the Node

Use this basic workflow:

```text
Manual Trigger → Zotero Library → Log
```

Recommended test sequence:

1. Test `collections` to verify authentication and the Library ID.
2. Test `search` with a broad term that exists in the library.
3. Copy a returned item's `key`.
4. Test `getItem` with that key.
5. Confirm that the Log node receives the expected formatted result.

For a public library, first try without `apiKey`. If Zotero returns `403`, add an API key with permission to read the library.

---

## Workflow Integration

### Search → Filter → Log

```text
Manual Trigger → Zotero Library (search) → Function/Filter → Log
```

Use this pattern to search Zotero, filter items by DOI or publication, and record the selected references.

### Input Key → Get Item → Summarizer

```text
Webhook/Form → Zotero Library (getItem) → AI Summarizer → Output
```

The webhook or form can provide a plain Item Key or an object such as:

```json
{
  "itemKey": "AB12CD34"
}
```

### Schedule → Search → Database

```text
Schedule → Zotero Library (search) → Loop → Database
```

Use this pattern to periodically synchronize recent Zotero references with another system.

### Common Patterns

- Search term → Zotero (`search`) → AI Summary — summarize matching literature
- Zotero (`search`) → Filter (`doi` exists) → Citation formatter — prepare a bibliography
- Item Key → Zotero (`getItem`) → Report generator — insert reference metadata into a report
- Zotero (`collections`) → Loop → Database — synchronize collection metadata

---

## Rate Limiting

The node waits at least 300 milliseconds between requests made by the same node instance, which targets approximately three requests per second.

If Zotero returns HTTP `429`:

1. The node reads the `Retry-After` response header.
2. It waits for that number of seconds.
3. If the header is absent, it waits five seconds.
4. It retries the request.

The implementation retries again whenever another `429` response is received; it does not currently enforce a maximum retry count.

---

## Error Handling

All operation errors are wrapped using this format:

```text
Zotero <operation> failed: <message>
```

### Missing Library ID

```text
Zotero search failed: Library ID is required
```

Raised when `libraryId` is empty.

### Missing Search Query

```text
Zotero search failed: Search query is required
```

Intended to be raised when neither configuration nor input provides a query. Note that the current implementation converts non-string input with `String(data)`; an `undefined` input may therefore become the literal query `"undefined"` instead of producing this error.

### Missing Item Key

```text
Zotero getItem failed: Item Key is required for getItem
```

Raised when no configured or incoming Item Key can be found.

### Zotero API Error

```text
Zotero <operation> failed: Zotero API Error (<status>): <response body>
```

Raised for non-success HTTP responses other than `429`.

### Unknown Operation

```text
Zotero <operation> failed: Unknown operation <operation>
```

This is a defensive runtime error. The schema normally prevents unsupported operation values.

---

## Troubleshooting

### `Zotero API Error (401)`

**Cause**

The API key is invalid, expired, revoked, or incorrectly copied.

**Solution**

Create or copy a valid key from Zotero API settings and verify that no extra spaces were included.

---

### `Zotero API Error (403)`

**Cause**

The selected library is private or the API key does not have permission to read it.

**Solution**

Grant the key read access to the correct User or Group library, and verify `libraryType` and `libraryId`.

---

### `Zotero API Error (404)`

**Cause**

The User ID, Group ID, or Item Key does not exist in the selected library, or `libraryType` is incorrect.

**Solution**

Confirm the numeric Library ID, choose the correct library type, and copy the Item Key from an actual item returned by `search`.

---

### Search Returns an Empty `results` Array

**Cause**

The query does not match any items, the selected filter is too restrictive, or the wrong library was selected.

**Solution**

- Try a broad title word known to exist in the library.
- Set `itemType` to `all`.
- Remove the `tag` filter.
- Try `searchMode: "everything"`.
- Verify the Library ID and library type.

---

### Search Unexpectedly Uses `undefined`

**Cause**

When `query` is empty and the node receives `undefined`, the current code can convert it to the literal string `"undefined"`.

**Solution**

Always configure `query` for standalone tests, or provide a valid string from the previous node. A code-level improvement would explicitly reject `null` and `undefined` before calling `String(data)`.

---

### `getItem` Cannot Find the Item

**Cause**

An Item Key from another library was used, or a title/DOI was supplied instead of the Zotero Item Key.

**Solution**

Run `search`, copy the returned `key`, and use that exact value with the same `libraryId` and `libraryType`.

---

### Only 100 Collections Are Returned

**Cause**

The `collections` operation uses a fixed `limit` of 100 and does not implement pagination.

**Solution**

For libraries with more than 100 collections, extend the node to follow Zotero pagination links or support `start`/pagination parameters.

---

## Security

- Store `apiKey` in a secrets manager or protected workflow variable.
- Do not paste live API keys into shared examples, logs, screenshots, or source control.
- Give the Zotero key only the permissions required by the workflow.
- Use a read-only key for this node because all implemented operations are read operations.
- Revoke and recreate a key immediately if it is exposed.
- Remember that returned Zotero records may contain private research notes, URLs, tags, or personal metadata.

---

## Notes and Limitations

- The node uses Zotero Web API version 3.
- All implemented operations are read-only.
- Search results are sorted by `date` in descending order.
- The returned `count` is the number of items in the current response, not the library's total matching-item count.
- Search item types are limited by the node schema to `journalArticle`, `book`, `thesis`, `conferencePaper`, and `all`.
- Search pagination is not implemented.
- Collection pagination is not implemented and the request limit is fixed at 100.
- The code does not validate the documented 8-character Item Key length before sending the request.
- Broader `everything` searches can be slower than `titleCreatorYear` searches.
- If Zotero returns an item without a `data` property, the node returns the raw item unchanged.
- Creator names are built from `name` or from `firstName` plus `lastName`.

---

## Related

- **Manual Trigger** — start a local Zotero test workflow
- **Function** — transform, filter, or format Zotero results
- **If** — route items based on type, tag, DOI, or other metadata
- **Loop** — process each search result or collection
- **Log** — inspect the node output during testing
- **HTTP Request** — call Zotero endpoints not implemented by this node
- **AI/Summarization nodes** — summarize abstracts or build literature reviews

---

## Changelog

### 1.0.0 — 2026-09-02
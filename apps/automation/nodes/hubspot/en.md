---
node_id: "hubspot"
title: "HubSpot"
description: "Read and manage HubSpot CRM records, associations, properties, and pipelines."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-12"
author: "Fusion Team"
tags: [integration, hubspot, crm, contacts, companies, deals, tickets]
related_nodes: []
---

<!-- SECTION: overview -->
# HubSpot

> **Category:** Peer-only Integrations | **Type:** Action Node

Use the HubSpot node to send one request to the HubSpot CRM API per execution. It covers contacts, companies, deals, tickets, line items, products, quotes, notes, tasks, engagements, associations, properties, and pipelines. The default operation lists contacts. The node returns the parsed JSON response without reshaping it.

This page describes the supplied node code and its generated requests. Availability of a particular CRM endpoint still depends on the HubSpot API and the token's permissions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

`operation` defaults to `getContacts`; the other fields are optional in the schema. The handler applies operation-specific checks at runtime. Fields with `dependsOn` are shown only for the listed operations in the configuration UI, although the handler may use them elsewhere.

| Parameter | Type | Default | Use |
|-----------|------|---------|-----|
| `operation` | enum | `getContacts` | Selects the request. See the operation tables below. |
| `accessToken` | string | None | OAuth or private app bearer token. Required at runtime for every operation; can also come from input. |
| `contactId`, `companyId`, `dealId`, `ticketId` | string | None | Source record ID for singular read, update, or delete operations and associations. |
| `lineItemId`, `productId`, `quoteId`, `noteId`, `taskId`, `engagementId` | string | None | Record ID for the corresponding singular operation. The first five can also identify association sources when `objectType` matches. |
| `engagementType` | enum | None | `calls`, `emails`, `meetings`, `notes`, or `tasks`. Required for create, update, and delete engagement operations. |
| `objectType` | string | `contacts` for properties; `deals` for pipelines | Object type in those URL paths. Required explicitly for associations. |
| `fromObjectId` | string | None | Source record ID for associations. Takes precedence over the ID field selected by `objectType`. |
| `toObjectType`, `toObjectId` | string | None | Target type and ID for associations. |
| `propertyName` | string | None | Property name for `updateProperty`. |
| `pipelineId` | string | None | Pipeline ID for `getPipeline`. |
| `searchQuery` | string | None | Required for search operations. Sent as the trimmed `query` value in the JSON body. |
| `objectIds` | string | None | Comma-separated IDs used to build `inputs: [{"id": ...}]` for batch read and batch delete. |
| `requestBody` | string containing a JSON object | None | Complete payload for create and update operations. Batch reads/deletes and searches construct their own bodies; default association creation has no body. |
| `limit` | number | None | Query parameter for list and association reads. |
| `after` | string | None | Paging token for list and association reads. |
| `properties` | string | None | Comma-separated property names for list, singular read, and batch read requests. |

The node sends requests to `https://api.hubapi.com`. It sets `Accept: application/json` and `User-Agent: FusionV2/1.0`, plus `Authorization: Bearer <accessToken>` using the token resolved for the current execution. It adds `Content-Type: application/json` for a POST or PATCH with a nonempty body. Keep the token in private workflow configuration or pass it through a protected input.

<!-- /SECTION: configuration -->

---

<!-- SECTION: operations -->
## Operations

The paths below are relative to `https://api.hubapi.com`. `X` represents one of `contacts`, `companies`, `deals`, or `tickets`; `XId` is its corresponding ID field. The same singular and list pattern also applies to the other object groups shown below.

### Contacts, companies, deals, and tickets

| Operation pattern | Method | Path | Notes |
|-------------------|--------|------|-------|
| Plural list, such as `getContacts` | GET | `/crm/v3/objects/X` | Adds `limit`, `after`, and `properties` when supplied. |
| Singular create, such as `createContact` | POST | `/crm/v3/objects/X` | Requires `requestBody`. |
| Singular read, such as `getContact` | GET | `/crm/v3/objects/X/{XId}` | Requires the ID; can add `properties`. |
| `updateX` (singular name) | PATCH | `/crm/v3/objects/X/{XId}` | Requires the ID and `requestBody`. |
| `deleteX` (singular name) | DELETE | `/crm/v3/objects/X/{XId}` | Requires the ID. |
| `batchReadX` | POST | `/crm/v3/objects/X/batch/read` | Builds an `inputs` array from `objectIds`; includes a `properties` array when supplied. |
| `batchCreateX` | POST | `/crm/v3/objects/X/batch/create` | Requires `requestBody`. |
| `batchUpdateX` | POST | `/crm/v3/objects/X/batch/update` | Requires `requestBody`. |
| `batchDeleteX` | POST | `/crm/v3/objects/X/batch/archive` | Builds an `inputs` array from `objectIds`. |
| `searchX` | POST | `/crm/v3/objects/X/search` | Sends `{ "query": "<trimmed searchQuery>" }`. |

The exact operation names for each group are:

| Object | Available operations |
|--------|----------------------|
| Contacts | `getContacts`, `createContact`, `getContact`, `updateContact`, `deleteContact`, `batchReadContacts`, `batchCreateContacts`, `batchUpdateContacts`, `batchDeleteContacts`, `searchContacts` |
| Companies | `getCompanies`, `createCompany`, `getCompany`, `updateCompany`, `deleteCompany`, `batchReadCompanies`, `batchCreateCompanies`, `batchUpdateCompanies`, `batchDeleteCompanies`, `searchCompanies` |
| Deals | `getDeals`, `createDeal`, `getDeal`, `updateDeal`, `deleteDeal`, `batchReadDeals`, `batchCreateDeals`, `batchUpdateDeals`, `batchDeleteDeals`, `searchDeals` |
| Tickets | `getTickets`, `createTicket`, `getTicket`, `updateTicket`, `deleteTicket`, `batchReadTickets`, `batchCreateTickets`, `batchUpdateTickets`, `batchDeleteTickets`, `searchTickets` |

### Other CRM objects

For each row, the node provides the listed operations. List and singular GET requests accept `properties`; list GET requests also accept `limit` and `after`. Create and update require `requestBody`; singular read, update, and delete require the corresponding ID.

| Object | Operations | Path |
|--------|------------|------|
| Line items | `getLineItems`, `createLineItem`, `getLineItem`, `updateLineItem`, `deleteLineItem` | `/crm/v3/objects/line_items[/{lineItemId}]` |
| Products | `getProducts`, `createProduct`, `getProduct`, `updateProduct`, `deleteProduct` | `/crm/v3/objects/products[/{productId}]` |
| Quotes | `getQuotes`, `createQuote`, `getQuote`, `updateQuote`, `deleteQuote` | `/crm/v3/objects/quotes[/{quoteId}]` |
| Notes | `getNotes`, `createNote`, `getNote`, `updateNote`, `deleteNote` | `/crm/v3/objects/notes[/{noteId}]` |
| Tasks | `getTasks`, `createTask`, `getTask`, `updateTask`, `deleteTask` | `/crm/v3/objects/tasks[/{taskId}]` |
| Engagements | `getEngagements`, `createEngagement`, `getEngagement`, `updateEngagement`, `deleteEngagement` | See below. |

The HTTP methods for these groups are GET for lists and singular reads, POST for creates, PATCH for updates, and DELETE for deletes. The bracketed ID suffix applies only to singular operations. For engagements, `getEngagements` uses `/crm/v3/objects/engagements` and `getEngagement` uses `/crm/v3/objects/engagements/{engagementId}`. Create, update, and delete instead use `/crm/v3/objects/{engagementType}` or `/crm/v3/objects/{engagementType}/{engagementId}`, with a required `engagementType` from `calls`, `emails`, `meetings`, `notes`, or `tasks`.

### Associations, properties, and pipelines

| Operation | Method | Path | Runtime requirements |
|-----------|--------|------|----------------------|
| `getAssociations` | GET | `/crm/v4/objects/{objectType}/{sourceId}/associations/{toObjectType}` | `objectType`, `toObjectType`, and source ID; optional `limit` and `after`. |
| `createAssociation` | PUT | `/crm/v4/objects/{objectType}/{sourceId}/associations/default/{toObjectType}/{toObjectId}` | Both types, source ID, and target ID; no request body. |
| `deleteAssociation` | DELETE | `/crm/v4/objects/{objectType}/{sourceId}/associations/{toObjectType}/{toObjectId}` | Both types, source ID, and target ID; no request body. |
| `getProperties` | GET | `/crm/v3/properties/{objectType}` | No additional runtime validation. |
| `updateProperty` | PATCH | `/crm/v3/properties/{objectType}/{propertyName}` | `propertyName` and `requestBody`. |
| `getPipelines` | GET | `/crm/v3/pipelines/{objectType}` | No additional runtime validation. |
| `getPipeline` | GET | `/crm/v3/pipelines/{objectType}/{pipelineId}` | `pipelineId`. |

For association paths, `sourceId` comes from `fromObjectId` when provided. Otherwise, the node selects the ID field matching `objectType`: `contactId`, `companyId`, `dealId`, `ticketId`, `lineItemId`, `productId`, `quoteId`, `noteId`, or `taskId`. Supply `fromObjectId` for any other source type. The handler rejects missing, blank, and literal `"undefined"` IDs before building these paths.

<!-- /SECTION: operations -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Direction | Description |
|------|-----------|-------------|
| `input` | Input | Triggers one selected operation. An incoming object can fill missing configuration fields. |
| `success` | Output | Parsed JSON response returned without transformation. |
| `error` | Output | Error path for validation, request, HTTP, or JSON parsing failures. |

For an object input, the handler fills unset fields from matching camelCase or snake_case keys. Token aliases are `accessToken`, `access_token`, `token`, and `apiKey`; search aliases are `searchQuery`, `search_query`, `query`, and `q`; batch ID aliases are `objectIds`, `object_ids`, and `ids`. `requestBody` can come from `requestBody`, `body`, or `data`; an object value is JSON-stringified. It also accepts the matching ID aliases and `fromObjectId`/`from_object_id`, `engagementType`/`engagement_type`, association, pipeline, property, and object type aliases. Configured values take precedence.

A string input is used as an ID only for selected singular contact, company, deal, and ticket operations. For reliable behavior, use an object input with the named ID field.

The node URL-encodes individual path segments and uses `URLSearchParams` for query strings. It does not fetch further pages automatically. On a successful HTTP 204 response it returns `{ "status": 204 }`; on any other successful empty response it returns `{ "status": <HTTP status> }`. A nonempty success body must contain valid JSON. Non-2xx errors include status, status text, and response text. The handler prefixes all failures with `HubSpot <operation> failed:`. There is no retry, timeout, or cancellation logic; `stop()` is a no-op.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### List contacts

Configure the node with a private token and optional paging and property fields:

```json
{
  "accessToken": "<your-access-token>",
  "operation": "getContacts",
  "limit": 25,
  "properties": "email,firstname,lastname"
}
```

Pass the returned paging token into `after` on a later execution to request another page.

### Create a contact

`requestBody` is a **string** containing JSON, not an object-valued configuration field:

```json
{
  "accessToken": "<your-access-token>",
  "operation": "createContact",
  "requestBody": "{\"properties\":{\"email\":\"alex@example.com\",\"firstname\":\"Alex\"}}"
}
```

Alternatively, configure `operation` and `accessToken` on the node, then send this object into `input`:

```json
{
  "data": {
    "properties": {
      "email": "alex@example.com",
      "firstname": "Alex"
    }
  }
}
```

The handler serializes `data` to `requestBody` when no configured body is present. Each execution sends a new create request.

### Search contacts

```json
{
  "accessToken": "<your-access-token>",
  "operation": "searchContacts",
  "searchQuery": "alex@example.com"
}
```

The node sends `{"query":"alex@example.com"}` as the POST body; `requestBody` is not used for searches.

### Read contacts in a batch

```json
{
  "accessToken": "<your-access-token>",
  "operation": "batchReadContacts",
  "objectIds": "101,102",
  "properties": "email,firstname"
}
```

This sends a POST body with `inputs: [{"id":"101"},{"id":"102"}]` and `properties: ["email","firstname"]`. Batch deletion uses the same `inputs` structure without `properties`.

### Create a default association

```json
{
  "accessToken": "<your-access-token>",
  "operation": "createAssociation",
  "objectType": "contacts",
  "fromObjectId": "101",
  "toObjectType": "companies",
  "toObjectId": "202"
}
```

This sends PUT to `/crm/v4/objects/contacts/101/associations/default/companies/202` with no body.

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Symptom | Explanation and action |
|---------|------------------------|
| `accessToken is required` | Supply a token in configuration or input. The resolved token is used in the bearer header. |
| `requestBody is required` or `Invalid JSON in requestBody` | Supply a nonempty JSON object string for create and update operations that require it. JSON arrays and scalar values are rejected. |
| `searchQuery is required` | Supply a nonblank search string. The node trims it and sends it in a `query` body field. |
| Authentication, permission, or rate-limit error | Check the supplied token, its granted permissions, and the service's rate limits. HTTP errors include the status and response text. |
| `objectIds is required` | Supply a comma-separated list without blank entries for batch read or delete. The node builds a POST `inputs` body. |
| `fromObjectId is required for this objectType` | Supply `fromObjectId` or the source ID field matching `objectType`. |
| `engagementType must be...` | Choose one of `calls`, `emails`, `meetings`, `notes`, or `tasks` for create, update, or delete engagement. |
| Pagination stops after one page | Read the API response's paging information and pass its next token as `after` in another execution. |

### Implementation limitations

The schema only displays `limit`, `after`, and `properties` for the core contact, company, deal, and ticket operations, although the handler also uses them for additional object lists, reads, and association reads. `getEngagements` and `getEngagement` still use the generic `engagements` path, while engagement writes use a concrete activity type path. Check the service API and scopes for these endpoints before relying on them.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-12 | Regenerated for the revised implementation: batch bodies, search queries, resolved bearer tokens, default associations, activity types, and empty response handling. |

<!-- /SECTION: changelog -->

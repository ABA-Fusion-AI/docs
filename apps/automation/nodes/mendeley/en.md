---
node_id: "mendeley-catalog"
title: "Mendeley Catalog"
description: "Search and retrieve academic papers from the Mendeley Catalog."
category: "Academic Research"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:

- mendeley
- academic
- research
- papers
- publications
- catalog
- doi
- pmid
- citations
- scholarly
- api
- oauth
- action

related_nodes:
- function
- if
- http-request

---

# Mendeley Catalog

> **Category:** academic-research-nodes | **Type:** Action Node

Search and retrieve academic publications from the **Mendeley Catalog API**.

The **Mendeley Catalog** node supports two operations:

- `search` — Search the Mendeley Catalog using a text query.
- `getById` — Retrieve a specific publication using a Mendeley Catalog ID or DOI.

The node authenticates with Mendeley using the OAuth 2.0 Client Credentials flow, caches the access token until expiration, applies a request delay for rate limiting, and retries once when an authenticated request receives a `401 Unauthorized` response.

### Supported Features

- Authenticate using Mendeley Client ID and Client Secret
- Search the Mendeley Catalog
- Retrieve publications by Mendeley Catalog ID
- Retrieve publications by DOI
- Filter searches by minimum publication year
- Limit search results
- Sort by relevance
- Sort by most recent
- Sort by most cited
- Retrieve citation and reader statistics
- Extract publication metadata
- Extract authors, abstracts, keywords, DOI, PMID, and ISBN
- Detect PDF attachment status
- Accept search queries from workflow input
- Accept document IDs or DOIs from workflow input
- Normalize Mendeley API responses
- Handle authentication and API errors
- Apply request rate limiting

### Use Cases

- Search academic literature
- Build research assistants
- Find scientific papers
- Build literature review workflows
- Find recent or highly cited research
- Retrieve paper metadata
- Feed papers into AI/RAG pipelines
- Store publications in databases
- Filter results using an `If` node
- Transform results using a `Function` node

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| ---------- | ---- | -------- | ------- | ----------- |
| `clientId` | `string` | ❌ No* | `""` | Mendeley OAuth Client ID. Required for API requests. |
| `clientSecret` | `string` | ❌ No* | `""` | Mendeley OAuth Client Secret. Required for API requests. |
| `operation` | `enum` | ❌ No | `"search"` | Operation: `search` or `getById`. |
| `query` | `string` | ❌ No* | — | Search query used by `search`. |
| `limit` | `number` | ❌ No* | `20` | Maximum search results. |
| `sort` | `enum` | ❌ No* | `"relevance"` | `relevance`, `most_recent`, or `most_cited`. |
| `minYear` | `number` | ❌ No* | — | Minimum publication year. |
| `documentId` | `string` | ❌ No* | — | Mendeley Catalog ID or DOI used by `getById`. |

\* These fields are optional in the schema but valid credentials or operation-specific values are required when executing the corresponding request.

### Configuration Schema

```typescript
const schema: SchemaTypeAny = v.object({
  clientId: v.string().optional().default(""),
  clientSecret: v.string().optional().default(""),

  operation: v.enum(["search", "getById"]).default("search"),

  query: v
    .string()
    .optional()
    .dependsOn({ eq: { field: "operation", value: "search" } }),

  limit: v
    .number()
    .optional()
    .default(20)
    .dependsOn({ eq: { field: "operation", value: "search" } }),

  sort: v
    .enum(["relevance", "most_recent", "most_cited"])
    .default("relevance")
    .dependsOn({ eq: { field: "operation", value: "search" } }),

  minYear: v
    .number()
    .optional()
    .metadata({ description: "Filter by minimum publication year" })
    .dependsOn({ eq: { field: "operation", value: "search" } }),

  documentId: v
    .string()
    .optional()
    .metadata({ description: "Mendeley Catalog ID or DOI" })
    .dependsOn({ eq: { field: "operation", value: "getById" } }),
});
```

---

## Operations

| Operation | Description |
| --------- | ----------- |
| `search` | Search the Mendeley Catalog using a text query. |
| `getById` | Retrieve a publication using a Mendeley Catalog ID or DOI. |

---

# Search

The `search` operation uses:

```text
https://api.mendeley.com/search/catalog
```

The request includes:

| Parameter | Description |
| --------- | ----------- |
| `query` | Search text. |
| `limit` | Maximum number of results. Defaults to `20`. |
| `view` | Set to `stats` to include statistics. |
| `min_year` | Minimum publication year. |
| `sort` | API sort field. |
| `order` | Sort direction. |

### Search Query

The configured `query` is preferred.

If no query is configured and workflow input is a string, that input is used.

Example input:

```text
machine learning
```

If neither is available, the node throws:

```text
Search query is required
```

### Search Sorting

`relevance` is the default and adds no sort parameters.

`most_recent` sends:

```text
sort=year
order=desc
```

`most_cited` sends:

```text
sort=citation_count
order=desc
```

### Minimum Year

Example:

```json
{
  "operation": "search",
  "query": "artificial intelligence",
  "minYear": 2024
}
```

The request contains:

```text
min_year=2024
```

---

## Search Output

The search operation returns:

```json
{
  "results": [],
  "count": 0
}
```

Each result contains:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | Mendeley Catalog ID. |
| `title` | `string` | Publication title. |
| `type` | `string` | Publication type. |
| `source` | `string` | Publication source. |
| `year` | `number` | Publication year. |
| `identifiers` | `object` | DOI, PMID, ISBN, and other identifiers when available. |
| `authors` | `string[]` | Normalized author names. |
| `abstract` | `string` | Abstract when available. |
| `link` | `string` | Publication link. |
| `pdfStatus` | `string` | `Attached` or `Not Available`. |
| `stats` | `object` | Reader and citation statistics. |
| `metadata` | `object` | DOI and ISBN metadata. |

### Search Output Example

```json
{
  "results": [
    {
      "id": "123456789",
      "title": "Example Research Paper",
      "type": "journal",
      "source": "Example Journal",
      "year": 2025,
      "identifiers": {
        "doi": "10.1234/example.2025"
      },
      "authors": [
        "John Smith",
        "Jane Doe"
      ],
      "abstract": "This paper presents an example research study.",
      "link": "https://www.mendeley.com/catalogue/example/",
      "pdfStatus": "Attached",
      "stats": {
        "readers": 125,
        "citations": 18
      },
      "metadata": {
        "doi": "10.1234/example.2025"
      }
    }
  ],
  "count": 1
}
```

The exact values depend on the Mendeley API response.

---

# Get By ID

The `getById` operation retrieves a publication using a Mendeley Catalog ID or DOI.

### Mendeley Catalog ID

For a normal ID, the node uses:

```text
catalog/{id}?view=all
```

### DOI

If the identifier starts with `10.` and contains `/`, it is treated as a DOI.

Example:

```text
10.1234/example.2025
```

The node uses:

```text
catalog?doi=10.1234%2Fexample.2025&view=all
```

The DOI is URL-encoded.

---

## Get By ID Input

`documentId` is preferred.

```json
{
  "operation": "getById",
  "documentId": "123456789"
}
```

If it is absent, workflow input can provide the identifier.

For string input:

```text
123456789
```

the string is used directly.

For object input, fields are checked in this order:

```text
id
documentId
doi
```

Example:

```json
{
  "doi": "10.1234/example.2025"
}
```

---

## Get By ID Output

The normalized document contains:

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | `string` | Mendeley Catalog ID. |
| `title` | `string` | Publication title. |
| `type` | `string` | Publication type. |
| `source` | `string` | Publication source. |
| `year` | `number` | Publication year. |
| `identifiers` | `object` | Publication identifiers. |
| `authors` | `string[]` | Normalized author names. |
| `abstract` | `string` | Abstract. |
| `link` | `string` | Publication link. |
| `keywords` | `string[]` | Keywords when available. |
| `stats` | `object` | Reader statistics. |
| `metadata` | `object` | DOI and PMID metadata. |

If no document is returned:

```text
null
```

### Output Example

```json
{
  "id": "123456789",
  "title": "Example Research Paper",
  "type": "journal",
  "source": "Example Journal",
  "year": 2025,
  "identifiers": {
    "doi": "10.1234/example.2025",
    "pmid": "12345678"
  },
  "authors": [
    "John Smith",
    "Jane Doe"
  ],
  "abstract": "This paper presents an example research study.",
  "link": "https://www.mendeley.com/catalogue/example/",
  "keywords": [
    "artificial intelligence",
    "machine learning"
  ],
  "stats": {
    "readers": 125
  },
  "metadata": {
    "doi": "10.1234/example.2025",
    "pmid": "12345678"
  }
}
```

---

## Authentication

The node uses:

```text
https://api.mendeley.com/oauth/token
```

The API base URL is:

```text
https://api.mendeley.com
```

Authentication uses the OAuth 2.0 Client Credentials flow.

The token request sends:

```text
grant_type=client_credentials
scope=all
```

The Client ID and Client Secret are encoded for HTTP Basic Authentication.

The request uses:

```text
Content-Type: application/x-www-form-urlencoded
```

A successful response is expected to contain:

```json
{
  "access_token": "ACCESS_TOKEN",
  "expires_in": 3600,
  "token_type": "bearer"
}
```

---

## Token Caching

The node caches the access token internally.

A token is reused while it remains valid with a 60-second safety buffer:

```typescript
Date.now() < this.tokenExpiration - 60000
```

When the token expires or approaches expiration, the node requests a new token.

---

## Authenticated Requests

Requests include:

```text
Authorization: Bearer <access_token>
```

and:

```text
Accept: application/vnd.mendeley-document.1+json
```

The authenticated request helper:

1. Applies the rate limit.
2. Obtains a valid access token.
3. Builds the API URL.
4. Adds the Bearer token.
5. Sends the request.
6. Handles `401 Unauthorized`.
7. Refreshes the token.
8. Retries once.
9. Throws errors for unsuccessful responses.
10. Parses the JSON response.

---

## Rate Limiting

The node applies a minimum delay of:

```text
200 ms
```

between requests.

This provides a local cap of approximately:

```text
5 requests per second
```

The actual Mendeley limits may vary by endpoint or account.

---

## Workflow Integration

### Search Papers

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "search",
        "query": "large language models",
        "limit": 20,
        "sort": "relevance"
      }
    }
  ]
}
```

### Search Recent Papers

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "search",
        "query": "AI healthcare",
        "limit": 10,
        "sort": "most_recent",
        "minYear": 2024
      }
    }
  ]
}
```

### Search Highly Cited Papers

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "search",
        "query": "machine learning",
        "limit": 10,
        "sort": "most_cited"
      }
    }
  ]
}
```

### Retrieve by ID

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "getById",
        "documentId": "123456789"
      }
    }
  ]
}
```

### Retrieve by DOI

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "getById",
        "documentId": "10.1234/example.2025"
      }
    }
  ]
}
```

### Input → Mendeley Search

```json
{
  "nodes": [
    {
      "id": "input",
      "type": "input"
    },
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "search",
        "query": "",
        "limit": 20,
        "sort": "relevance"
      }
    }
  ]
}
```

If the input is:

```text
graph neural networks
```

the node searches for that text.

### Input → Get By ID

```json
{
  "nodes": [
    {
      "id": "input",
      "type": "input"
    },
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "getById"
      }
    }
  ]
}
```

If the input is:

```text
10.1234/example.2025
```

the node retrieves the publication by DOI.

### Mendeley → Function

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "search",
        "query": "retrieval augmented generation",
        "limit": 10
      }
    },
    {
      "id": "process-papers",
      "type": "function"
    }
  ]
}
```

### Mendeley → If

```json
{
  "nodes": [
    {
      "id": "mendeley",
      "type": "mendeley-catalog",
      "config": {
        "clientId": "YOUR_CLIENT_ID",
        "clientSecret": "YOUR_CLIENT_SECRET",
        "operation": "search",
        "query": "artificial intelligence",
        "limit": 20
      }
    },
    {
      "id": "filter-papers",
      "type": "if"
    }
  ]
}
```

### Common Patterns

- Input → Mendeley → Function
- Input → Mendeley → If
- Mendeley → Function → Database
- Mendeley → Function → RAG Pipeline
- Mendeley → If → Notification
- Mendeley → Function → Research Report
- Mendeley → HTTP Request → Additional Processing

---

## Processing Details

### Author Normalization

An API author such as:

```json
{
  "first_name": "John",
  "last_name": "Smith"
}
```

becomes:

```text
John Smith
```

using:

```typescript
`${a.first_name} ${a.last_name}`.trim()
```

If no authors are returned:

```json
[]
```

is returned.

### PDF Status

If:

```text
file_attached = true
```

the node returns:

```text
"Attached"
```

Otherwise:

```text
"Not Available"
```

The node does not download the PDF.

### Identifiers

The original `identifiers` object is preserved.

Selected identifiers are also exposed in `metadata`.

Search metadata:

```json
{
  "doi": "10.1234/example",
  "isbn": "9780000000000"
}
```

Get-by-ID metadata:

```json
{
  "doi": "10.1234/example",
  "pmid": "12345678"
}
```

---

## Error Handling

All errors are wrapped using:

```text
Mendeley <operation> failed: <error message>
```

### Missing Credentials

```text
Mendeley Client ID and Client Secret are required.
```

### Authentication Failure

```text
Mendeley Auth failed: <status> <statusText>
```

### API Failure

```text
Mendeley API Request failed: <status> <statusText>
```

### Retry Failure

After a `401` and token refresh:

```text
Mendeley API Request failed after retry: <statusText>
```

### Missing Search Query

```text
Mendeley search failed: Search query is required
```

### Missing Document ID

```text
Mendeley getById failed: Document ID or DOI is required for getById
```

### Unknown Operation

```text
Mendeley <operation> failed: Unknown operation <operation>
```

---

## Troubleshooting

### Search Returns No Results

Try broader keywords, increase `limit`, remove `minYear`, or use `relevance` sorting.

### Recent Papers Are Missing

Check the `minYear` filter and ensure it is not excluding the required publications.

### Citation Sorting Looks Unexpected

Citation statistics and sorting behavior depend on the data available through the Mendeley API.

### Rate Limit Problems

The node applies a 200 ms local delay, but Mendeley may enforce additional endpoint or account-specific limits.

### Credentials Are Rejected

Verify the Client ID, Client Secret, OAuth configuration, and API access permissions.

---

## Security

`clientId` and `clientSecret` are credentials and should be treated as secrets.

Do not expose the Client Secret in workflow outputs, logs, or downstream nodes.

Access tokens are stored internally and are not returned in the node output.

Example:

```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET"
}
```

Use secure credential storage when supported by the workflow platform.

---

## Notes

The node returns normalized publication data rather than the complete raw Mendeley API response.

The node does not:

- Download PDFs
- Extract full-text content
- Generate summaries
- Generate embeddings
- Store publications
- Verify scientific claims
- Provide investment or academic advice

It is intended to retrieve and structure academic publication metadata for downstream workflow processing.

---

## Related

- [Function](./function.md) – Transform and process academic publication results
- [If](./if.md) – Filter and route publications
- [HTTP Request](./http-request.md) – Make additional HTTP requests

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-11 | Initial release |

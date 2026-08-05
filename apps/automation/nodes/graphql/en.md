---
node_id: "graphql"
title: "GraphQL"
description: "Execute GraphQL queries and mutations against any GraphQL API endpoint"
category: "utilities"
subcategory: "network"
version: "1.0.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - utility
  - graphql
  - api
  - query
  - mutation
related_nodes:
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# GraphQL

> **Category:** Utilities | **Type:** Action Node

Execute GraphQL queries and mutations against compatible GraphQL API endpoints using GET or POST requests.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GraphQL** node sends GraphQL queries and mutations to a GraphQL API endpoint. It supports JSON and GraphQL request formats, variables, operation names, API keys, custom headers, and configurable response formats.

### Key Features

- **Queries and Mutations:** Execute GraphQL read and write operations
- **GET and POST Requests:** Use either supported HTTP method
- **Request Formats:** Send JSON or plain GraphQL requests
- **Variables:** Pass GraphQL variables as valid JSON
- **Operation Names:** Execute a specific named operation
- **Authentication:** Add a bearer API key or custom headers
- **Response Formats:** Return JSON data or a serialized string
- **Dynamic Input:** Read query information from incoming workflow data
- **Error Handling:** Route request and GraphQL errors to the error output

### Use Cases

- Retrieve data from GraphQL APIs
- Execute GraphQL mutations
- Query services using dynamic variables
- Integrate third-party GraphQL platforms
- Automate GraphQL API operations
- Transform GraphQL responses with downstream nodes

### Dynamic Input

The node can receive GraphQL request information from incoming data.

| Field | Type | Description |
|-------|------|-------------|
| `query` | `string` | Overrides the configured GraphQL query |
| `variables` | `string` or `object` | Overrides the configured GraphQL variables |
| `operationName` | `string` | Overrides the configured operation name |
| `headers` | `object` | Adds or overrides request headers |

A direct string input can also be used as the GraphQL query.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `endpointUrl` | `url` | ✅ Yes | — | GraphQL API endpoint URL |
| `httpMethod` | `enum` | ✅ Yes | `POST` | HTTP request method: `GET` or `POST` |
| `requestFormat` | `enum` | ✅ Yes | `json` | Request format: `json` or `graphql` |
| `responseFormat` | `enum` | ✅ Yes | `json` | Response format: `json` or `string` |
| `query` | `string` | ❌ No | — | GraphQL query or mutation |
| `variables` | `string` | ❌ No | — | GraphQL variables as a valid JSON string |
| `operationName` | `string` | ❌ No | — | Name of the GraphQL operation to execute |
| `apiKey` | `string` | ❌ No | — | Bearer API key added to the authorization header |
| `headers` | `array` | ❌ No | `[]` | Additional custom request headers |

> Although the `query` configuration parameter is optional, a GraphQL query is required at execution time. It can be provided through the node configuration, incoming data, or a direct string input.

### JSON Request Format

When `requestFormat` is set to `json`, the node sends a JSON request body.

```json
{
  "query": "query GetCountry($code: ID!) { country(code: $code) { name } }",
  "variables": {
    "code": "MA"
  },
  "operationName": "GetCountry"
}
```

The request uses:

```text
Content-Type: application/json
```

### GraphQL Request Format

When `requestFormat` is set to `graphql`, the query is sent as a plain GraphQL request body.

```graphql
query {
  countries {
    code
    name
  }
}
```

The request uses:

```text
Content-Type: application/graphql
```

### Variables

The `variables` parameter must contain valid JSON.

```json
{
  "code": "MA"
}
```

Invalid example:

```text
{ code: MA }
```

### Custom Headers

Custom headers use a list of key-value pairs.

```json
[
  {
    "key": "X-Client-Id",
    "value": "workflow-client"
  },
  {
    "key": "X-Environment",
    "value": "staging"
  }
]
```

### API Key Authentication

When `apiKey` is configured, the node adds the following request header:

```text
Authorization: Bearer <apiKey>
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data containing a query, variables, operation name, or headers |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `string` | Successful GraphQL response |
| `error` | `object` | Request, validation, parsing, network, or GraphQL execution error |

### JSON Success Output

When `responseFormat` is set to `json`, the output contains the GraphQL response data.

```json
{
  "success": true,
  "data": {
    "country": {
      "name": "Morocco",
      "capital": "Rabat"
    }
  },
  "errors": null,
  "extensions": null
}
```

### String Success Output

When `responseFormat` is set to `string`, the GraphQL response is returned as a serialized string.

### Error Output

```json
{
  "success": false,
  "error": "GraphQL errors: Cannot query field \"invalidField\" on type \"Country\".",
  "query": "query { country(code: \"MA\") { invalidField } }"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Query a Country

Retrieve information about Morocco.

**Configuration:**

```text
Endpoint URL: https://countries.trevorblades.com/
HTTP Method: POST
Request Format: json
Response Format: json
```

**Query:**

```graphql
query {
  country(code: "MA") {
    name
    capital
    currency
    emoji
  }
}
```

**Output:**

```json
{
  "success": true,
  "data": {
    "country": {
      "name": "Morocco",
      "capital": "Rabat",
      "currency": "MAD",
      "emoji": "🇲🇦"
    }
  },
  "errors": null,
  "extensions": null
}
```

---

### Example: Query with Variables

Execute a named query using variables.

**Query:**

```graphql
query GetCountry($code: ID!) {
  country(code: $code) {
    name
    capital
    currency
  }
}
```

**Variables:**

```json
{
  "code": "MA"
}
```

**Operation Name:**

```text
GetCountry
```

---

### Example: Dynamic Query from Input

A previous node can provide the query dynamically.

**Input:**

```json
{
  "query": "query { countries { code name } }"
}
```

The incoming `query` overrides the query configured directly in the node.

---

### Example: Dynamic Variables

A previous node can provide query variables dynamically.

**Input:**

```json
{
  "query": "query GetCountry($code: ID!) { country(code: $code) { name capital } }",
  "variables": {
    "code": "MA"
  },
  "operationName": "GetCountry"
}
```

---

### Example: Custom Headers

Send additional request headers.

**Configuration:**

```json
[
  {
    "key": "X-Client-Id",
    "value": "fusion-workflow"
  }
]
```

---

### Example: API Key Authentication

Configure the API key field with a valid token.

The node sends:

```text
Authorization: Bearer <apiKey>
```

---

### Example: Mutation

Execute a GraphQL mutation.

```graphql
mutation CreateUser($name: String!, $email: String!) {
  createUser(name: $name, email: $email) {
    id
    name
    email
  }
}
```

**Variables:**

```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

### Example: Invalid Variables

Invalid variables configuration:

```text
{ code: MA }
```

**Error Output:**

```json
{
  "success": false,
  "error": "Invalid JSON in variables field"
}
```

---

### Example: Invalid GraphQL Field

**Query:**

```graphql
query {
  country(code: "MA") {
    invalidField
  }
}
```

**Error Output:**

```json
{
  "success": false,
  "error": "GraphQL errors: Cannot query field \"invalidField\" on type \"Country\"."
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Execute a GraphQL request and inspect the response
```

### Sample Workflow: GraphQL Country Query

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger"
    },
    {
      "id": "graphql-request",
      "type": "graphql",
      "config": {
        "endpointUrl": "https://countries.trevorblades.com/",
        "httpMethod": "POST",
        "requestFormat": "json",
        "responseFormat": "json",
        "query": "query { country(code: \"MA\") { name capital currency } }"
      }
    },
    {
      "id": "show-result",
      "type": "log"
    }
  ]
}
```

### Common Patterns

- **Simple Query:** Manual Trigger → GraphQL → Log
- **Dynamic Query:** Previous Node → GraphQL → Function
- **API Integration:** Trigger → GraphQL → Database
- **Mutation Flow:** Webhook → GraphQL Mutation → Notification
- **Response Transformation:** GraphQL → Function → Next Action

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Node is not registered

**Cause:** The GraphQL node package is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package containing the `graphql` node was built, deployed, and loaded successfully.

#### GraphQL query is required

**Cause:** No query was provided through the node configuration or incoming data.

**Solution:** Configure the `query` parameter or pass a query from the previous node.

#### Invalid JSON in variables field

**Cause:** The variables parameter does not contain valid JSON.

**Solution:** Use quoted JSON property names and valid JSON values.

```json
{
  "code": "MA"
}
```

#### GraphQL request failed

**Cause:** The endpoint returned a non-successful HTTP response.

**Solution:** Verify the endpoint URL, method, authentication, headers, and request body.

#### GraphQL errors returned

**Cause:** The server accepted the HTTP request but rejected the GraphQL query or mutation.

**Solution:** Verify field names, arguments, variables, operation names, and the target GraphQL schema.

#### Unauthorized request

**Cause:** The endpoint requires authentication or the supplied API key is invalid.

**Solution:** Configure a valid API key or custom authorization header.

#### Unexpected response format

**Cause:** The endpoint returned content that does not match the selected response format.

**Solution:** Verify the endpoint response and select the correct response format.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `GraphQL query is required` | Missing query | Configure or pass a GraphQL query |
| `Invalid JSON in variables field` | Invalid variables JSON | Correct the JSON syntax |
| `GraphQL request failed` | HTTP request failure | Check endpoint and request settings |
| `GraphQL errors` | Invalid GraphQL operation | Check the GraphQL schema |
| `HTTP 401` | Invalid or missing authentication | Configure valid credentials |
| `HTTP 404` | Incorrect endpoint URL | Verify the API endpoint |
| `Network error` | Endpoint unavailable | Check connectivity and endpoint status |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) - Send generic HTTP requests
- [Function](./function.md) - Transform GraphQL response data
- [Log](./log.md) - Inspect GraphQL output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-05 | Initial release |

<!-- /SECTION: changelog -->
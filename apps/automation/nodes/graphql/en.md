---
node_id: "graphql"
title: "GraphQL"
description: "Execute GraphQL queries and mutations against any GraphQL API endpoint"
category: "utilities"
subcategory: "network"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
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
- **Response Formats:** Return structured JSON data or a serialized string
- **Dynamic Input:** Read query information from incoming workflow data
- **Error Handling:** Return structured error information for HTTP, network, and GraphQL execution failures

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
| `apiKey` | `string` | ❌ No | — | Bearer API key added to the Authorization header |
| `headers` | `array` | ❌ No | `[]` | Additional custom request headers |

> Although the `query` configuration parameter is optional, a GraphQL query is required at execution time. It can be provided through the node configuration, incoming data, a direct string input, or the endpoint URL query parameter.

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

When `requestFormat` is set to `graphql`, the query is sent directly as the request body.

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

> The target GraphQL server must support the `application/graphql` content type. Some GraphQL endpoints only accept JSON-formatted requests.

### GET Requests

When `httpMethod` is set to `GET`, the query is URL-encoded and appended to the endpoint as the `query` parameter.

For example:

```text
https://example.com/graphql?query=<encoded-query>
```

The node does not send a request body for GET requests.

> For GET requests, the current implementation only appends the GraphQL `query` to the URL. Configured `variables` and `operationName` are not included in the GET request.

### Variables

The `variables` configuration parameter must contain valid JSON.

```json
{
  "code": "MA"
}
```

Invalid example:

```text
{ code: MA }
```

Invalid JSON in the variables field is rejected before the HTTP request is executed.

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

Custom headers are added after the default `Content-Type` and API key headers.

Incoming `headers` data can also add or override request headers at execution time.

### API Key Authentication

When `apiKey` is configured, the node adds the following request header:

```text
Authorization: Bearer <apiKey>
```

Custom or incoming headers can also be used when the target API requires another authentication mechanism.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data containing a query, variables, operation name, or headers |

The incoming data can override configured values for `query`, `variables`, `operationName`, and request headers.

A direct string input can also be used as the GraphQL query.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `string` | Successful response or structured HTTP/GraphQL execution failure |
| `error` | `object` | Node-level errors such as invalid variables JSON |

### JSON Success Output

When `responseFormat` is set to `json`, a successful request returns an object containing the extracted GraphQL data and the raw server response.

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
  "extensions": null,
  "raw": {
    "data": {
      "country": {
        "name": "Morocco",
        "capital": "Rabat"
      }
    }
  }
}
```

### String Success Output

When `responseFormat` is set to `string`, the complete GraphQL response is serialized using `JSON.stringify()`.

Example:

```text
{"data":{"country":{"name":"Germany","capital":"Berlin","currency":"EUR"}}}
```

### GraphQL or Request Failure Output

HTTP request failures, network failures, and GraphQL errors handled during request execution return a structured result.

```json
{
  "success": false,
  "error": "GraphQL errors: Cannot query field \"invalidField\" on type \"Country\".",
  "query": "{ country(code: \"MA\") { invalidField } }"
}
```

### Node-Level Error

Errors that occur before the request execution block, such as invalid JSON in the `variables` field, are raised as node errors.

Example:

```text
Invalid JSON in variables field
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
      "currency": "MAD"
    }
  },
  "errors": null,
  "extensions": null,
  "raw": {
    "data": {
      "country": {
        "name": "Morocco",
        "capital": "Rabat",
        "currency": "MAD"
      }
    }
  }
}
```

---

### Example: GET Request

Execute a GraphQL query using the GET method.

**Configuration:**

```text
Endpoint URL: https://countries.trevorblades.com/
HTTP Method: GET
Request Format: json
Response Format: json
```

**Query:**

```graphql
{
  country(code: "FR") {
    name
    capital
    currency
  }
}
```

The query is URL-encoded and sent through the `query` URL parameter.

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
  "code": "IT"
}
```

**Operation Name:**

```text
GetCountry
```

**Output Data:**

```json
{
  "country": {
    "name": "Italy",
    "capital": "Rome",
    "currency": "EUR"
  }
}
```

---

### Example: String Response

Return the GraphQL response as a serialized string.

**Configuration:**

```text
HTTP Method: POST
Request Format: json
Response Format: string
```

Example output:

```text
{"data":{"country":{"name":"Germany","capital":"Berlin","currency":"EUR"}}}
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

Incoming variables can be provided either as an object or as a JSON string.

---

### Example: Custom Headers

Send additional request headers.

**Configuration:**

```json
[
  {
    "key": "X-Test-Source",
    "value": "Fusion-GraphQL"
  }
]
```

Custom headers are included in the GraphQL HTTP request.

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

The exact mutation fields depend on the schema exposed by the target GraphQL API.

---

### Example: Invalid Variables

Invalid variables configuration:

```text
{"code":"MA"
```

**Error:**

```text
Invalid JSON in variables field
```

Invalid variables JSON is rejected before the HTTP request is executed.

---

### Example: Invalid GraphQL Field

**Query:**

```graphql
{
  country(code: "MA") {
    invalidField
  }
}
```

**Output:**

```json
{
  "success": false,
  "error": "GraphQL errors: Cannot query field \"invalidField\" on type \"Country\".",
  "query": "{ country(code: \"MA\") { invalidField } }"
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

**Cause:** No query was provided through the node configuration, incoming data, direct string input, or endpoint URL.

**Solution:** Configure the `query` parameter or pass a query from the previous node.

#### Invalid JSON in variables field

**Cause:** The variables parameter does not contain valid JSON.

**Solution:** Use quoted JSON property names and valid JSON values.

```json
{
  "code": "MA"
}
```

This validation occurs before the GraphQL HTTP request is executed.

#### GraphQL request failed

**Cause:** The endpoint returned a non-successful HTTP response.

**Solution:** Verify the endpoint URL, HTTP method, authentication, headers, request format, and request body.

#### GraphQL errors returned

**Cause:** The server accepted the HTTP request but returned one or more GraphQL errors.

**Solution:** Verify field names, arguments, variables, operation names, and the target GraphQL schema.

#### GraphQL request format rejected

**Cause:** The target endpoint may not support requests using the `application/graphql` content type.

**Solution:** If the endpoint does not support plain GraphQL request bodies, set `requestFormat` to `json`.

#### Unauthorized request

**Cause:** The endpoint requires authentication or the supplied credentials are invalid.

**Solution:** Configure a valid API key or provide the required authentication headers.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `GraphQL query is required` | Missing query | Configure or pass a GraphQL query |
| `Invalid JSON in variables field` | Invalid variables JSON | Correct the JSON syntax |
| `GraphQL request failed` | HTTP request failure | Check endpoint and request settings |
| `GraphQL errors` | GraphQL execution or validation error | Check the GraphQL schema and operation |
| `HTTP 401` | Invalid or missing authentication | Configure valid credentials |
| `HTTP 404` | Incorrect endpoint URL | Verify the API endpoint |
| Network-related error | Endpoint unavailable or network failure | Check connectivity and endpoint availability |

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
| 1.0.0 | 2026-09-01 | Initial documentation and validated workflow example |

<!-- /SECTION: changelog -->
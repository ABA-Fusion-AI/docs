---
node_id: graphql
title: GraphQL
description: Execute GraphQL queries and mutations against any GraphQL API endpoint.
category: Integration
subcategory: API
version: 1.0.0
language: en
author: ABA Fusion AI
last_updated: 2026-08-05
---

# GraphQL

The **GraphQL** node allows you to execute GraphQL queries and mutations against any GraphQL endpoint.

It supports custom HTTP headers, API keys, variables, operation names and multiple request/response formats.

---

## Inputs

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Endpoint URL | String | ✅ | GraphQL endpoint URL. |
| HTTP Method | Select | ✅ | HTTP method used to execute the request (POST or GET). |
| Request Format | Select | ✅ | Request payload format. |
| Response Format | Select | ✅ | Expected response format. |
| Query | String | ✅ | GraphQL query or mutation. |
| Variables | JSON | ❌ | Variables used by the query. |
| Operation Name | String | ❌ | GraphQL operation name. |
| API Key | Password | ❌ | API key if required by the endpoint. |
| Headers | JSON | ❌ | Additional HTTP headers. |

---

## Outputs

### Success

Returned when the GraphQL request is executed successfully.

Example:

```json
{
  "success": true,
  "data": {
    "country": {
      "name": "Morocco",
      "capital": "Rabat",
      "currency": "MAD"
    }
  }
}
```

---

### Error

Returned when the request fails or GraphQL returns one or more errors.

Example:

```json
{
  "success": false,
  "error": "GraphQL errors: Cannot query field \"invalidField\" on type \"Country\"."
}
```

---

# Example

Endpoint

```
https://countries.trevorblades.com/
```

Query

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

Expected Result

```json
{
  "data": {
    "country": {
      "name": "Morocco",
      "capital": "Rabat",
      "currency": "MAD",
      "emoji": "🇲🇦"
    }
  }
}
```

---

## Common Use Cases

- Execute GraphQL queries.
- Execute GraphQL mutations.
- Retrieve data from GraphQL APIs.
- Send variables dynamically.
- Authenticate using API keys or custom headers.
- Integrate GraphQL services into Fusion AI workflows.

---

## Notes

- Ensure the endpoint is reachable.
- Variables must be valid JSON.
- GraphQL errors are returned through the **Error** output.
- Sensitive credentials should be provided through secure parameters.
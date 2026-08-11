---
node_id: "medicaments-api-fr"
title: "Médicaments API FR"
description: "Access French medications data from the BDPM (Base de Données Publique des Médicaments) via the Médicaments API."
category: "API"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - medicaments
  - medications
  - france
  - bdpm
  - api
  - healthcare
  - medicine
  - search
  - generics
  - action
related_nodes:
  - function
  - if
  - http-request
---

# Médicaments API FR

> **Category:** api-nodes | **Type:** Action Node

Access French medications data from the **BDPM (Base de Données Publique des Médicaments)** through the `medicaments-api.giygas.dev` API.

The **Médicaments API FR** node allows workflows to retrieve the medication database, browse paginated database results, search for medications, retrieve a medication by its CIS identifier, search generic medications, retrieve generic groups, and check API health status.

The node automatically handles URL encoding, HTTP errors, and rate-limit information returned by the API.

### Supported Operations

- Get Database
- Get Database Page
- Search Medicament
- Get Medicament by ID
- Search Generiques
- Get Generique Group
- Get Health

### Use Cases

- Search French medications
- Retrieve medication information by CIS identifier
- Browse the French public medication database
- Find generic medications
- Retrieve generic medication groups
- Build medication lookup workflows
- Validate API availability
- Monitor API rate limits
- Integrate French medication data into automated workflows

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `baseUrl` | `string` | ❌ No | `https://medicaments-api.giygas.dev` | Base URL of the Médicaments API. |
| `operation` | `string` | ❌ No | `searchMedicament` | API operation to perform. |
| `page` | `number` | Required for `getDatabasePage` | — | Database page number. Must be an integer greater than or equal to 1. |
| `element` | `string` | Required for `searchMedicament` | — | Medication search term. Must contain between 3 and 50 characters. |
| `cis` | `number` | Required for `getMedicamentById` | — | CIS identifier of the medication. Must be between 1 and 999999999. |
| `libelle` | `string` | Required for `searchGeneriques` | — | Label or search term used to find generic medications. |
| `groupId` | `number` | Required for `getGeneriqueGroup` | — | Generic group identifier. Must be an integer greater than or equal to 1. |

### Supported Operations

| Operation | Description |
| --------- | ----------- |
| `getDatabase` | Retrieves the medication database. |
| `getDatabasePage` | Retrieves a specific page of the medication database. |
| `searchMedicament` | Searches for medications using a text search term. |
| `getMedicamentById` | Retrieves a medication using its CIS identifier. |
| `searchGeneriques` | Searches for generic medications using a label. |
| `getGeneriqueGroup` | Retrieves a generic medication group by its identifier. |
| `getHealth` | Checks whether the Médicaments API is healthy and available. |

---

## Operation Details

### Get Database

Retrieves the medication database from the API.

**Endpoint**

~~~text
GET /database
~~~

**Configuration**

~~~json
{
  "operation": "getDatabase"
}
~~~

---

### Get Database Page

Retrieves a specific page from the medication database.

**Endpoint**

~~~text
GET /database/{page}
~~~

**Configuration**

~~~json
{
  "operation": "getDatabasePage",
  "page": 1
}
~~~

The `page` value must be an integer greater than or equal to `1`.

---

### Search Medicament

Searches the medication database using a text term.

**Endpoint**

~~~text
GET /medicament/{element}
~~~

The search term is automatically URL encoded before the request is sent.

**Configuration**

~~~json
{
  "operation": "searchMedicament",
  "element": "paracetamol"
}
~~~

The `element` parameter must contain between 3 and 50 characters.

---

### Get Medicament by ID

Retrieves a medication using its **CIS identifier**.

**Endpoint**

~~~text
GET /medicament/id/{cis}
~~~

**Configuration**

~~~json
{
  "operation": "getMedicamentById",
  "cis": 123456789
}
~~~

The `cis` value must be an integer between `1` and `999999999`.

---

### Search Generiques

Searches for generic medications using a label or search term.

**Endpoint**

~~~text
GET /generiques/{libelle}
~~~

The search term is automatically URL encoded before the request is sent.

**Configuration**

~~~json
{
  "operation": "searchGeneriques",
  "libelle": "paracetamol"
}
~~~

---

### Get Generique Group

Retrieves a generic medication group using its group identifier.

**Endpoint**

~~~text
GET /generiques/group/{groupId}
~~~

**Configuration**

~~~json
{
  "operation": "getGeneriqueGroup",
  "groupId": 123
}
~~~

The `groupId` must be an integer greater than or equal to `1`.

---

### Get Health

Checks the health and availability of the Médicaments API.

**Endpoint**

~~~text
GET /health
~~~

**Configuration**

~~~json
{
  "operation": "getHealth"
}
~~~

---

## Inputs & Outputs

### Inputs

This node does not require workflow input. All parameters are provided through node configuration.

Parameters are conditionally required depending on the selected operation:

| Operation | Required Parameter |
| --------- | ------------------ |
| `getDatabase` | None |
| `getDatabasePage` | `page` |
| `searchMedicament` | `element` |
| `getMedicamentById` | `cis` |
| `searchGeneriques` | `libelle` |
| `getGeneriqueGroup` | `groupId` |
| `getHealth` | None |

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Indicates whether the API request completed successfully. |
| `operation` | `string` | Operation that was executed. |
| `data` | `object \| array` | Data returned by the Médicaments API. |
| `rateLimit` | `object` | Rate-limit information returned by the API. |
| `statusCode` | `number` | HTTP response status code. |
| `error` | `Error` | Returned if validation, network, or API operations fail. |

### Rate Limit Information

The node extracts the following HTTP response headers:

| Property | Header | Description |
| -------- | ------ | ----------- |
| `limit` | `X-RateLimit-Limit` | Maximum number of requests allowed. |
| `remaining` | `X-RateLimit-Remaining` | Number of requests remaining. |
| `rate` | `X-RateLimit-Rate` | Rate-limit value reported by the API. |

If a header is missing or cannot be parsed as a number, its value is returned as `null`.

### Output Example

~~~json
{
  "success": true,
  "operation": "searchMedicament",
  "data": [
    {
      "cis": 123456789,
      "name": "Paracetamol"
    }
  ],
  "rateLimit": {
    "limit": 100,
    "remaining": 95,
    "rate": 100
  },
  "statusCode": 200
}
~~~

The exact structure of the `data` property depends on the API operation and the response returned by the Médicaments API.

---

## Workflow Integration

### Sample Workflow: Search for a Medication

~~~json
{
  "nodes": [
    {
      "id": "search-medicament",
      "type": "medicaments-api-fr",
      "config": {
        "operation": "searchMedicament",
        "element": "paracetamol"
      }
    }
  ]
}
~~~

### Sample Workflow: Retrieve Medication by CIS

~~~json
{
  "nodes": [
    {
      "id": "get-medicament",
      "type": "medicaments-api-fr",
      "config": {
        "operation": "getMedicamentById",
        "cis": 123456789
      }
    }
  ]
}
~~~

### Sample Workflow: Check API Health

~~~json
{
  "nodes": [
    {
      "id": "health-check",
      "type": "medicaments-api-fr",
      "config": {
        "operation": "getHealth"
      }
    }
  ]
}
~~~

### Common Patterns

- Form → Search Medicament
- Search Medicament → Function → Process Results
- CIS Identifier → Get Medicament by ID
- Search Generiques → Process Results
- Get Database Page → Function → Transform Data
- Health Check → If → Continue or Alert
- API Request → Rate Limit Check → Continue Workflow

---

## Troubleshooting

### `page` is required

**Cause**

The `getDatabasePage` operation was selected without providing a page number.

**Solution**

Provide a valid integer in the `page` parameter.

Example:

~~~json
{
  "operation": "getDatabasePage",
  "page": 1
}
~~~

---

### `element` is required

**Cause**

The `searchMedicament` operation was selected without providing a search term.

**Solution**

Provide a search term containing between 3 and 50 characters.

Example:

~~~json
{
  "operation": "searchMedicament",
  "element": "paracetamol"
}
~~~

---

### `cis` is required

**Cause**

The `getMedicamentById` operation was selected without providing a CIS identifier.

**Solution**

Provide a valid CIS identifier.

Example:

~~~json
{
  "operation": "getMedicamentById",
  "cis": 123456789
}
~~~

---

### `libelle` is required

**Cause**

The `searchGeneriques` operation was selected without providing a search label.

**Solution**

Provide a value for `libelle`.

Example:

~~~json
{
  "operation": "searchGeneriques",
  "libelle": "paracetamol"
}
~~~

---

### `groupId` is required

**Cause**

The `getGeneriqueGroup` operation was selected without providing a generic group identifier.

**Solution**

Provide a valid integer greater than or equal to `1`.

Example:

~~~json
{
  "operation": "getGeneriqueGroup",
  "groupId": 123
}
~~~

---

### HTTP Error

**Cause**

The Médicaments API returned a non-success HTTP status code.

**Solution**

Check the API response and verify that the configured `baseUrl` is correct.

When the API returns a JSON error containing a `message` property, that message is included in the node error.

The node reports errors using the following format:

~~~text
Médicaments API error: <error message>
~~~

---

### Invalid Base URL

**Cause**

The configured `baseUrl` is invalid or the API cannot be reached.

**Solution**

Verify the `baseUrl` configuration.

Default:

~~~text
https://medicaments-api.giygas.dev
~~~

---

### Rate Limit Reached

**Cause**

The API may reject requests after the configured rate limit has been exceeded.

**Solution**

Inspect the `rateLimit` output to monitor the API's reported limits:

~~~json
{
  "rateLimit": {
    "limit": 100,
    "remaining": 0,
    "rate": 100
  }
}
~~~

Consider adding an `If` or delay/retry mechanism to your workflow when the remaining request count is low.

---

### API Health Check Failed

**Cause**

The Médicaments API may be unavailable or experiencing an outage.

**Solution**

Use the `getHealth` operation to verify API availability before performing dependent operations.

---

## API URL

The default API base URL is:

~~~text
https://medicaments-api.giygas.dev
~~~

The `baseUrl` parameter can be overridden when using another compatible API endpoint.

---

## Related

- [Function](./function.md) – Transform and process medication API data
- [If](./if.md) – Route workflows based on API results
- [HTTP Request](./http-request.md) – Integrate with other external APIs

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-10 | Initial release |
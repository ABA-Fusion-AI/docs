---
node_id: "gleif"
title: "Global LEI Index (GLEIF)"
description: "Search and retrieve Legal Entity Identifier (LEI) data from the GLEIF API."
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags:
  - gleif
  - lei
  - legal-entity
  - company
  - finance
  - accounting
  - compliance
  - api
related_nodes:
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# Global LEI Index (GLEIF)

> **Category:** Business & Commerce | **Subcategory:** Finance & Accounting | **Type:** Action Node

Search and retrieve Legal Entity Identifier (LEI) records for organizations from the Global Legal Entity Identifier Foundation (GLEIF) API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Global LEI Index (GLEIF)** node connects workflows to the GLEIF API. It helps identify legal entities and retrieve standardized LEI reference data, including organization names, registration information, entity status, and relationship information when available.

### Key Features

- **Entity Search:** Find legal entities using supported search criteria
- **LEI Lookup:** Retrieve a record for a known Legal Entity Identifier
- **Standardized Data:** Use globally standardized legal-entity reference information
- **Relationship Data:** Access parent and child relationship information when published
- **Workflow Ready:** Pass results to Function, Restructure, storage, reporting, or validation nodes
- **Error Routing:** Send lookup, request, and API errors to the error output
- **Progress Visibility:** Supports running status while the GLEIF request is processed

### Typical Use Cases

- Verify a company’s legal identity before onboarding
- Enrich customer, supplier, or counterparty records
- Validate LEIs for financial or compliance workflows
- Search entities by name, country, or registration information
- Retrieve entity data for accounting and reporting systems
- Check parent or child entity relationships

### What Is an LEI?

A Legal Entity Identifier is a globally recognized identifier for legal entities participating in financial transactions. An LEI record can connect an entity to reference information such as its legal name, jurisdiction, registration identifier, status, and reporting relationships.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Search and Lookup Criteria

The node accepts search or lookup criteria supported by its configuration and incoming workflow data. Common criteria include:

| Criterion | Type | Description |
|-----------|------|-------------|
| `lei` | `string` | Exact 20-character LEI to retrieve |
| `name` | `string` | Legal-entity name or name fragment to search for |
| `country` | `string` | Country or jurisdiction filter, typically using an ISO country code |
| `registrationId` | `string` | Registration identifier issued by a local registration authority |
| `page` | `number` | Page number for a multi-result search, when pagination is enabled |
| `pageSize` | `number` | Number of results requested per page, when pagination is enabled |

Use the exact field names exposed by the node configuration. If a value is supplied through the `input` connection, it can be used as dynamic search data when supported by the workflow runtime.

### LEI Format

An LEI is normally represented as a 20-character alphanumeric string:

```text
5493001KJTIIGC8Y1R12
```

Preserve the complete value, including all letters and numbers. Do not replace an LEI with a stock ticker, company name, or local registration number when performing an exact LEI lookup.

### Dynamic Input

A preceding node can provide search criteria dynamically:

```json
{
  "name": "Example Corporation",
  "country": "US"
}
```

For an exact lookup, the input can contain:

```json
{
  "lei": "5493001KJTIIGC8Y1R12"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `any` | Optional incoming search criteria or LEI lookup data |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | GLEIF entity record or search response |
| `error` | `object` | Error details when the request or lookup fails |

### Success Response

A successful response can contain one or more legal-entity records. Depending on the lookup or search performed, records may include fields such as:

```json
{
  "lei": "5493001KJTIIGC8Y1R12",
  "legalName": "Example Corporation",
  "entityStatus": "ACTIVE",
  "jurisdiction": "US",
  "registrationId": "123456789",
  "registrationAuthority": "Example authority",
  "lastUpdateDate": "2026-01-15"
}
```

Search responses may additionally include result counts, pagination information, and a list of matching records. Use a Function or Restructure node when a downstream system requires a smaller or fixed schema.

### Error Response Example

```json
{
  "success": false,
  "error": "GLEIF request failed",
  "details": "No matching legal entity was found"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Lookup by LEI

Retrieve the record associated with a known LEI.

**Input:**

```json
{
  "lei": "5493001KJTIIGC8Y1R12"
}
```

Use the success output to inspect the legal name, status, jurisdiction, and other available reference data.

---

### Example 2: Search by Legal Name

Search for entities by name.

**Input:**

```json
{
  "name": "Example Corporation"
}
```

If multiple entities match, inspect the returned records and use jurisdiction or registration information to select the correct entity.

---

### Example 3: Filter by Name and Country

Narrow a search to a jurisdiction.

**Input:**

```json
{
  "name": "Example Corporation",
  "country": "US"
}
```

Country and jurisdiction values must use the format expected by the node and the GLEIF API.

---

### Example 4: Enrich a Supplier Record

Pass supplier information from a previous node and use GLEIF to find standardized legal-entity data.

**Input:**

```json
{
  "supplierName": "Example Corporation",
  "country": "US"
}
```

After the lookup, use Restructure or Function to combine the supplier’s internal identifier with the returned LEI and entity status.

---

### Example 5: Inspect Results During Development

Connect the GLEIF success output to a Log node. Logging the complete response helps identify the available field names before configuring filters, mappings, or storage steps.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search the Global LEI Index and inspect entity data
```

### Common Patterns

- **Exact Lookup:** Manual Trigger → GLEIF → Log
- **Supplier Enrichment:** Supplier Data → GLEIF → Restructure → Storage
- **Compliance Check:** Entity Input → GLEIF → Function → Decision
- **Entity Search:** Search Criteria → GLEIF → Filter → Notification
- **Batch Enrichment:** Iterator → GLEIF → Merge → Report

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No matching entity was found

**Cause:** The name, LEI, country, or registration identifier does not match a published GLEIF record.

**Solution:** Verify the spelling and identifier, broaden the name search, or confirm the jurisdiction filter.

#### The LEI is rejected

**Cause:** The value is incomplete, contains invalid characters, or is not a valid 20-character LEI.

**Solution:** Copy the complete LEI and preserve its exact characters.

#### Too many search results

**Cause:** The search criterion is too broad, such as a short name fragment without a country filter.

**Solution:** Add jurisdiction, registration, or other supported criteria and use pagination when available.

#### Unexpected response shape

**Cause:** Exact lookups and multi-result searches can return different response structures.

**Solution:** Connect a Log node first, inspect the complete response, and then map the required fields with Function or Restructure.

#### GLEIF request fails

**Cause:** The API is unavailable, the request is malformed, or the workflow environment cannot reach the service.

**Solution:** Verify the criteria, retry safely, and check network access and service availability.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Invalid LEI` | LEI format is invalid | Use the complete 20-character LEI |
| `No matching entity` | Search returned no records | Verify or broaden the search criteria |
| `Invalid search criteria` | Unsupported or malformed input | Use the fields and formats supported by the node |
| `GLEIF request failed` | Network or API failure | Check connectivity and retry safely |
| `Unexpected response format` | Returned data differs from the expected shape | Log and inspect the full response |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](../function/en.md) — Filter or transform GLEIF entity data
- [Restructure](../restructure/en.md) — Map entity records into a downstream schema
- [Merge](../merge/en.md) — Combine results from multiple entity searches
- Log — Inspect GLEIF responses during workflow development

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-19 | Initial documentation for the Global LEI Index (GLEIF) node |

<!-- /SECTION: changelog -->

---
node_id: "clinical-tables-search"
title: "Clinical Tables NLM Search"
description: "Search NLM Clinical Tables databases for medical codes, terms, drugs, genes, conditions, and related clinical data."
category: "healthcare-life-sciences"
subcategory: "clinical-databases"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - healthcare
  - clinical
  - medical-codes
  - nlm
  - icd
  - drugs
  - genes
related_nodes:
  - clinical-trials-search
  - chembl
  - biorxiv
---

<!-- SECTION: header -->
# Clinical Tables NLM Search

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Search National Library of Medicine Clinical Tables databases for medical codes, conditions, drug ingredients, genes, procedures, units, and other clinical reference data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Clinical Tables NLM Search** node queries the NLM Clinical Tables API and returns structured clinical reference data.

It supports multiple medical and scientific databases, including ICD classifications, HCPCS, Human Phenotype Ontology, LOINC items, procedures, conditions, drug ingredients, RxTerms, UCUM, COSMIC, disease names, and genes.

The node automatically builds the API request, retries temporary failures, validates the response, and converts the returned data into a consistent result structure.

### Key Features

- **Multiple Clinical Databases:** Search medical codes, conditions, drugs, genes, procedures, and scientific reference data.
- **Structured Results:** Returns normalized names and optional codes.
- **Configurable Result Limit:** Request between 1 and 500 results.
- **Metadata Support:** Include or omit the database description.
- **Retry Handling:** Retries temporary network failures, server errors, and rate limits.
- **Timeout Protection:** Uses a 30-second request timeout.
- **Automatic Validation:** Validates search terms, database selection, and response structure.
- **No Credentials Required:** Uses the public NLM Clinical Tables API.

### Supported Databases

| Database | Description | Returns Codes |
|----------|-------------|---------------|
| `icd10cm` | ICD-10-CM clinical diagnosis codes | Yes |
| `hcpcs` | Healthcare Common Procedure Coding System | Yes |
| `hpo` | Human Phenotype Ontology | Yes |
| `icd11_codes` | ICD-11 classification codes | Yes |
| `icd9cm_dx` | ICD-9-CM diagnosis codes | Yes |
| `icd9cm_sg` | ICD-9-CM procedure codes | Yes |
| `loinc_items` | LOINC medical forms and questions | No |
| `procedures` | Major surgeries and implants | No |
| `conditions` | Clinical conditions and synonyms | No |
| `drug_ingredients` | Prescribable drug ingredients | No |
| `rxterms` | Drug names and strengths | No |
| `ucum` | Unified Code for Units of Measure | No |
| `cosmic` | Somatic mutation and cancer genomics data | No |
| `disease_names` | Genetic disease names from ClinVar | No |
| `genes` | Gene symbols and regions | No |

### Use Cases

- Search ICD diagnosis codes
- Find clinical condition names
- Retrieve drug ingredient names
- Search genes and disease names
- Look up procedure terminology
- Retrieve phenotype ontology terms
- Build healthcare search interfaces
- Enrich clinical workflow data
- Validate medical terminology
- Create medical reference tools

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `database` | `enum` | ✅ Yes | — | Clinical Tables database to search. |
| `terms` | `string` | ✅ Yes | — | Search text. Must contain at least one character and cannot exceed 500 characters. |
| `maxResults` | `number` | ❌ No | `50` | Maximum number of results to request. Accepts values from `1` to `500`. |
| `includeMetadata` | `boolean` | ❌ No | `true` | Include the database description in the output. |

### Database Values

The `database` parameter accepts:

```text
icd10cm
hcpcs
hpo
icd11_codes
icd9cm_dx
icd9cm_sg
loinc_items
procedures
conditions
drug_ingredients
rxterms
ucum
cosmic
disease_names
genes
```

### Default Values

| Parameter | Default |
|-----------|---------|
| `maxResults` | `50` |
| `includeMetadata` | `true` |

### Search Term Rules

The node validates the search text before sending the request.

- Leading and trailing spaces are removed.
- Empty search terms are rejected.
- Search terms longer than 500 characters are rejected.

### Result Limit Handling

The node normalizes `maxResults` before the request:

- Values below `1` fall back to `50`.
- Values above `500` are limited to `500`.
- Invalid numeric values fall back to `50`.

### Metadata Behavior

When `includeMetadata` is enabled, the output contains the database description.

When disabled, `database_description` is returned as an empty string.

### Request Behavior

The node:

- sends requests to the NLM Clinical Tables API;
- uses a 30-second timeout;
- retries temporary failures up to three times;
- applies exponential backoff;
- retries HTTP `429` and server errors;
- returns empty results for HTTP `404`;
- validates the API response structure.

<!-- /SECTION: configuration -->
---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow data. The node uses the configured parameters to perform the search. |

The node primarily uses the configured parameters.


### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `database` | `string` | Database used for the search. |
| `database_description` | `string` | Human-readable database description, or an empty string when metadata is disabled. |
| `search_terms` | `string` | Normalized search terms. |
| `total_count` | `number` | Total number of matches reported by the API. |
| `returned_results` | `number` | Number of processed results returned by the node. |
| `has_codes` | `boolean` | Indicates whether results from the selected database include codes. |
| `results` | `array` | Normalized clinical search results. |

### Result Structure with Codes

Databases such as `icd10cm`, `hcpcs`, and `hpo` return results containing a code and name.

```json
{
  "database": "icd10cm",
  "database_description": "ICD-10-CM (International Classification of Diseases, 10th Revision, Clinical Modification)",
  "search_terms": "diabetes mellitus",
  "total_count": 25,
  "returned_results": 5,
  "has_codes": true,
  "results": [
    {
      "code": "E11.9",
      "name": "Type 2 diabetes mellitus without complications"
    }
  ]
}
```

### Result Structure without Codes

Databases such as `conditions`, `procedures`, and `drug_ingredients` return names without codes.

```json
{
  "database": "conditions",
  "database_description": "Clinical conditions with synonyms (Regenstrief Institute derivative)",
  "search_terms": "diabetes",
  "total_count": 12,
  "returned_results": 5,
  "has_codes": false,
  "results": [
    {
      "name": "Diabetes mellitus"
    }
  ]
}
```

### Empty Results

When no matches are found, the node returns a valid response with an empty result array.

```json
{
  "database": "conditions",
  "database_description": "Clinical conditions with synonyms (Regenstrief Institute derivative)",
  "search_terms": "unknown-clinical-term",
  "total_count": 0,
  "returned_results": 0,
  "has_codes": false,
  "results": []
}
```

### Metadata Disabled

When `includeMetadata` is `false`, the database description is returned as an empty string.

```json
{
  "database": "conditions",
  "database_description": "",
  "search_terms": "diabetes",
  "total_count": 12,
  "returned_results": 5,
  "has_codes": false,
  "results": [
    {
      "name": "Diabetes mellitus"
    }
  ]
}
```

### Error Response

The node throws an error when validation or the API request fails.

Examples:

```text
Search terms are required and cannot be empty
```

```text
Search terms cannot exceed 500 characters
```

```text
Error searching Clinical Tables (conditions): <error message>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Search Clinical Conditions

Search for clinical condition names.

**Configuration**

```text
Database: conditions
Terms: diabetes
Max Results: 5
Include Metadata: true
```

**Output**

```json
{
  "database": "conditions",
  "database_description": "Clinical conditions with synonyms (Regenstrief Institute derivative)",
  "search_terms": "diabetes",
  "total_count": 12,
  "returned_results": 5,
  "has_codes": false,
  "results": [
    {
      "name": "Diabetes mellitus"
    }
  ]
}
```

---

### Example: Search ICD-10-CM Codes

Search diagnosis codes.

**Configuration**

```text
Database: icd10cm
Terms: diabetes mellitus
Max Results: 10
Include Metadata: true
```

The result contains ICD-10-CM codes and names.

---

### Example: Search Human Phenotype Ontology

Search phenotype terms.

**Configuration**

```text
Database: hpo
Terms: muscle weakness
Max Results: 10
Include Metadata: true
```

---

### Example: Search Procedures

Search major procedures or implants.

**Configuration**

```text
Database: procedures
Terms: hip replacement
Max Results: 10
Include Metadata: true
```

---

### Example: Search Drug Ingredients

Search prescribable drug ingredients.

**Configuration**

```text
Database: drug_ingredients
Terms: aspirin
Max Results: 10
Include Metadata: true
```

---

### Example: Search RxTerms

Search drug names and strengths.

**Configuration**

```text
Database: rxterms
Terms: amoxicillin
Max Results: 10
Include Metadata: true
```

---

### Example: Search UCUM Units

Search standardized units of measure.

**Configuration**

```text
Database: ucum
Terms: milligram
Max Results: 10
Include Metadata: true
```

---

### Example: Search Disease Names

Search genetic disease names.

**Configuration**

```text
Database: disease_names
Terms: cystic fibrosis
Max Results: 10
Include Metadata: true
```

---

### Example: Search Genes

Search gene symbols and regions.

**Configuration**

```text
Database: genes
Terms: BRCA1
Max Results: 10
Include Metadata: true
```

---

### Example: Limit Results

Request only a small number of results.

**Configuration**

```text
Database: conditions
Terms: diabetes
Max Results: 3
Include Metadata: true
```

The node requests at most three items from the API.

---

### Example: Disable Metadata

Return results without the database description.

**Configuration**

```text
Database: conditions
Terms: diabetes
Max Results: 5
Include Metadata: false
```

The output contains:

```json
{
  "database_description": ""
}
```

---


### Example: Empty Search Terms

If no search terms are configured or received, the node throws:

```text
Search terms are required and cannot be empty
```

---

### Example: Search Terms Too Long

If the search query exceeds 500 characters, the node throws:

```text
Search terms cannot exceed 500 characters
```

<!-- /SECTION: examples -->
---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Clinical Tables Database
```

### Common Patterns

- **Clinical Condition Search:** Manual Trigger → Clinical Tables NLM Search → Log
- **Medical Code Lookup:** HTTP Request → Clinical Tables NLM Search → Log
- **Drug Ingredient Search:** Manual Trigger → Clinical Tables NLM Search → Log
- **Gene Lookup:** HTTP Request → Clinical Tables NLM Search → Log
- **Clinical Workflow:** Clinical Tables NLM Search → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "Search terms are required and cannot be empty"

**Cause**

No search query was configured or received from the previous workflow node.

**Solution**

Provide a non-empty value in the `terms` parameter or pass a string from a previous node.

---

#### "Search terms cannot exceed 500 characters"

**Cause**

The search query is longer than the supported limit.

**Solution**

Reduce the search text to fewer than 500 characters.

---

#### No Results Returned

**Cause**

The selected database contains no matching entries.

Example:

```json
{
  "total_count": 0,
  "returned_results": 0,
  "results": []
}
```

**Solution**

- Verify the selected database.
- Try a broader search term.
- Check the spelling.
- Use official clinical terminology when possible.

---

#### Empty Database Description

**Cause**

The `includeMetadata` option is disabled.

**Solution**

Enable `includeMetadata` if the database description is required.

---

#### API Timeout

**Cause**

The Clinical Tables API did not respond before the configured timeout.

**Solution**

Retry the request or reduce the requested number of results.

---

#### Temporary Server Error

**Cause**

The remote API returned a temporary server error.

The node automatically retries transient failures using exponential backoff.

**Solution**

Retry later if the error persists.

---

### Error Messages

| Error | Description |
|-------|-------------|
| `Search terms are required and cannot be empty` | No search query was provided. |
| `Search terms cannot exceed 500 characters` | The search query is too long. |
| `Error searching Clinical Tables (...)` | The Clinical Tables request failed. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- Clinical Trials Search
- ChEMBL
- BioRxiv

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release of the Clinical Tables NLM Search documentation. |

<!-- /SECTION: changelog -->
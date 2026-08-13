---
node_id: "census-gov"
title: "Census.gov"
description: "Get US Census data from the Census.gov API."
category: "Web Search & Information"
subcategory: "Government"
version: "1.0.0"
language: "en"
last_updated: "2026-08-13"
author: "Fusion Team"
tags:
  - census
  - government
  - demographics
  - usa
  - statistics
  - api
  - data
related_nodes:
  - open-data-search
  - world-bank
  - federal-register
---

<!-- SECTION: header -->
# Census.gov

> **Category:** Web Search & Information | **Type:** Action Node

Get US Census data from the Census.gov API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Census.gov** node queries the official US Census API to retrieve demographic, economic, housing, and geographic statistics. It is useful for workflows that need official public data for research, reporting, planning, and enrichment.

### Key Features

- **Official US Data:** Pull data directly from the Census.gov API
- **Demographic Insights:** Access population, race, age, education, and household statistics
- **Geographic Queries:** Filter data by state, county, tract, metro area, and more
- **Structured Output:** Return data in a machine-readable JSON structure
- **Dataset Discovery:** Query available tables and variables through the API
- **Workflow Integration:** Use results in dashboards, reports, comparisons, and enrichment tasks

### Use Cases

- Pull demographic data for a region
- Compare population statistics across geographies
- Enrich CRM or marketing data with public census insights
- Build reporting dashboards using official US statistics
- Support research or planning workflows with trusted government data

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `dataset` | `string` | ✅ Yes | — | Census dataset identifier such as `acs/acs5` or `cbp` |
| `variables` | `string` or `array` | ✅ Yes | — | One or more requested variables |
| `geo` | `string` | ✅ Yes | — | Geographic filter for the query, such as `state:*` or `county:*` |
| `year` | `number` | ❌ No | Latest supported | Census year to query |
| `apiKey` | `string` | ❌ No | — | Optional API key if required by the selected dataset |

### Example

```text
dataset: "acs/acs5"
variables: ["B01003_001E","B19013_001E"]
geo: "state:*"
year: 2022
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Query values or a structured object containing dataset, variables, and geo filters |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Census API response containing matching records |
| `error` | `object` | Error details if the request is invalid or the API fails |

### Success Output Example

```json
{
  "data": [
    {
      "state": "01",
      "B01003_001E": "5024279",
      "B19013_001E": "59400"
    }
  ],
  "source": "census.gov"
}
```

### Error Output Example

```json
{
  "success": false,
  "error": "The Census API request was invalid or returned no data.",
  "dataset": "acs/acs5"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Get Population and Median Household Income by State

```text
dataset: "acs/acs5"
variables: ["B01003_001E", "B19013_001E"]
geo: "state:*"
year: 2022
```

**Result:**

```json
{
  "data": [
    {
      "state": "01",
      "B01003_001E": "5024279",
      "B19013_001E": "59400"
    }
  ]
}
```

### Example: Government Data Enrichment Workflow

Use the node to enrich customer, region, or planning data with official demographic statistics from the US Census API.

<!-- /SECTION: examples -->

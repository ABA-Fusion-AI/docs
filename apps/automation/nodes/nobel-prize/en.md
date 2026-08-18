---
node_id: "nobel-prize"
title: "Nobel Prize"
description: "Get Nobel Prize laureate data from Nobel Prize API."
category: "Web Search & Information"
subcategory: "Nobel Prize"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - nobel-prize
  - laureate
  - nobel-foundation
  - research
  - academic
  - achievement
  - award
related_nodes:
  - google-scholar
  - science-direct-search
  - random-user
  - http-request
---

<!-- SECTION: header -->
# Nobel Prize

> **Category:** Web Search & Information | **Type:** Action Node

Get Nobel Prize laureate data from the official Nobel Prize API, including winners, achievements, and prize information by year or category.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Nobel Prize** node retrieves comprehensive data about Nobel Prize laureates, prizes, and winners from the official Nobel Foundation API. It allows workflows to search for and access detailed information about Nobel Prize winners across categories and years.

The node integrates with the Nobel Prize API to fetch laureate profiles, prize details, achievement descriptions, and award years for research, academic, and informational workflows.

### Key Features

- **Laureate Search:** Retrieve Nobel Prize winners by year, category, or name
- **Prize Categories:** Access data across all Nobel Prize categories (Physics, Chemistry, Physiology or Medicine, Literature, Peace, Economic Sciences)
- **Year Filtering:** Query winners for specific years or date ranges
- **Detailed Profiles:** Get comprehensive laureate information including biography, achievements, and affiliations
- **Structured JSON Output:** Returns machine-readable laureate records with standardized metadata
- **No API Key Required:** Uses the public Nobel Prize API

### Nobel Prize Categories

The API supports the following prize categories:

| Code | Category |
|------|----------|
| `physics` | Nobel Prize in Physics |
| `chemistry` | Nobel Prize in Chemistry |
| `medicine` | Nobel Prize in Physiology or Medicine |
| `literature` | Nobel Prize in Literature |
| `peace` | Nobel Peace Prize |
| `economic` | Nobel Memorial Prize in Economic Sciences |

### Typical Use Cases

- Research historical Nobel Prize winners
- Build timelines of achievements by field
- Create educational dashboards about Nobel laureates
- Analyze Nobel Prize distribution by country or year
- Enrich academic workflows with prize winner information
- Track achievements and breakthroughs by prize category

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `year` | `number` | ❌ No | — | Nobel Prize year to query (e.g., `2021`, `2020`, `2019`) |
| `category` | `string` | ❌ No | — | Prize category filter (e.g., `physics`, `chemistry`, `medicine`, `literature`, `peace`, `economic`) |
| `sortOrder` | `enum` | ❌ No | `asc` | Sort order for results: `asc` (ascending) or `desc` (descending) |
| `limit` | `number` | ❌ No | `10` | Maximum number of laureates to return |

### Category Reference

| Category | Description |
|----------|-------------|
| `physics` | Physics research and discovery |
| `chemistry` | Chemistry research and discovery |
| `medicine` | Physiology or medicine research |
| `literature` | Literary achievement and contribution |
| `peace` | Peace and international diplomacy |
| `economic` | Economic sciences |

### Example Configuration

```text
year: 2021
category: "physics"
limit: 10
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `number`, `string`, or `object` | Year value (e.g., `2021`) or object with query parameters like `year`, `category`, or `limit` |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Nobel Prize API response with laureate records and metadata |
| `error` | `object` | Error details when the request fails or no results are found |

### Success Response Example

```json
{
  "prizes": [
    {
      "year": "2021",
      "category": "physics",
      "laureates": [
        {
          "id": "1234",
          "firstname": "Syukuro",
          "surname": "Manabe",
          "motivation": "for the physical modelling of Earth's climate, quantifying variability and reliably predicting global warming",
          "share": "1/4",
          "affiliation": {
            "name": "Princeton University",
            "city": "Princeton",
            "country": "USA"
          }
        },
        {
          "id": "1235",
          "firstname": "Klaus",
          "surname": "Hasselmann",
          "motivation": "for the development of probabilistic models of Earth's climate and quantifying its variability",
          "share": "1/4",
          "affiliation": {
            "name": "Max Planck Institute for Meteorology",
            "city": "Hamburg",
            "country": "Germany"
          }
        }
      ]
    }
  ]
}
```

### Laureate Structure

Each laureate object in the response contains:

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique laureate identifier |
| `firstname` | `string` | First name of the laureate |
| `surname` | `string` | Surname of the laureate |
| `motivation` | `string` | Brief description of the achievement or contribution |
| `share` | `string` | Prize share (e.g., `1/1`, `1/2`, `1/4`) |
| `affiliation` | `object` | Organization, city, and country of affiliation |
| `born` | `string` | Birth date (YYYY-MM-DD format, when available) |
| `died` | `string` | Death date (YYYY-MM-DD format, when available) |
| `bornCountry` | `string` | Country of birth |
| `bornCountryCode` | `string` | ISO country code |

### Error Response Example

```json
{
  "success": false,
  "error": "No Nobel Prize data found for the specified year or category"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Get Physics Prize Winners for 2021

```text
year: 2021
category: "physics"
```

**Result:**

```json
{
  "prizes": [
    {
      "year": "2021",
      "category": "physics",
      "laureates": [
        {
          "firstname": "Syukuro",
          "surname": "Manabe",
          "motivation": "for the physical modelling of Earth's climate..."
        }
      ]
    }
  ]
}
```

---

### Example 2: Get All Prize Categories for a Year

```text
year: 2020
limit: 50
```

### Example 3: Nobel Prize Timeline Workflow

Use the node to build a historical timeline of Nobel Prize winners, then pass results to visualization or dashboard nodes.

```text
For Each Year (1901-2021)
  → Nobel Prize (year)
  → Extract laureates by category
  → Log or Store in Database
```

### Example 4: Nobel Prize Analysis Workflow

Retrieve Nobel Prize data and analyze distribution by country or institution.

```text
Manual Trigger
  → Nobel Prize (year: 2021, category: "medicine")
  → Group by affiliation country
  → Create summary statistics
  → Generate report
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Get Nobel Prize laureate data
```

### Common Patterns

- **Historical Analysis:** Loop through years to build Nobel Prize distribution over time
- **Category Comparison:** Query each category separately to compare achievement rates
- **Geographic Distribution:** Group results by laureate country or affiliation location
- **Educational Content:** Extract and format laureate bios for educational platforms

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: api-reference -->
## API Reference

### Nobel Prize API Endpoints

The node accesses the official Nobel Prize API:

```text
Base URL: https://api.nobelprize.org/2/
Endpoints:
  - /prizes?year={year}&category={category}
  - /laureates?year={year}
```

### Rate Limiting

- No API key required
- Standard public API rate limits apply
- Recommended: Cache results when querying the same year/category multiple times

<!-- /SECTION: api-reference -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No results returned for a year

**Cause:** The specified year may not have had a Nobel Prize ceremony (e.g., some years skipped during wars).

**Solution:** Check the official Nobel Prize website for prize ceremonies in that year.

#### API connection timeout

**Cause:** The Nobel Prize API server is temporarily unavailable or slow to respond.

**Solution:** Retry the request or reduce the number of results with the `limit` parameter.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `No data found for year` | Year has no prize data | Verify the year is valid and had a ceremony |
| `Invalid category` | Category parameter is incorrect | Check supported category codes |
| `API connection failed` | Network or server issue | Retry after a few minutes |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Google Scholar](../google-scholar/en.md) — Academic research and publication search
- [Science Direct Search](../science-direct-search/en.md) — Scientific article search
- [Random User](../random-user/en.md) — Generate test data
- [HTTP Request](../http-request/en.md) — Custom API queries

<!-- /SECTION: related -->

---

<!-- SECTION: security -->
## Security

The node uses the public Nobel Prize API with no authentication required. No sensitive credentials need to be stored. Ensure your server has outbound internet connectivity to access the API.

<!-- /SECTION: security -->

---
node_id: "world-bank"
title: "World Bank Data"
description: "Retrieve global development indicators, country lists, topics, and data sources from the World Bank API."
category: "Web Search & Information"
subcategory: "Economic & Public Data"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - world-bank
  - economic-data
  - development
  - gdp
  - population
  - indicators
  - public-data
  - statistics
related_nodes:
  - census-gov
  - geo-names
  - currency-converter
  - log
---

<!-- SECTION: header -->
# World Bank Data

> **Category:** Web Search & Information | **Subcategory:** Economic & Public Data | **Type:** Action Node

Access global development data, country metadata, indicator catalogs, topic lists, and data-source information through the World Bank API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **World Bank Data** node provides workflow access to the World Bank's public API. It supports time-series indicator data as well as reference resources that help identify countries, indicators, topics, and source datasets.

### Supported Resources

- **`data`:** Retrieve an indicator for one country or a group of countries over a year or date range
- **`countries`:** Retrieve country and regional metadata, including ISO codes, income level, lending type, and coordinates
- **`indicators`:** Browse or inspect available development indicators
- **`topics`:** Browse high-level topics to which indicators are mapped
- **`sources`:** List World Bank data sources and dataset metadata

### Key Features

- **Development Indicators:** Retrieve GDP, population, poverty, education, health, trade, and other statistics
- **Country Filtering:** Query a specific country such as Morocco (`MA`) or all countries
- **Date Ranges:** Request a year or range such as `2020:2024`
- **Pagination:** Control the number of returned records with `perPage`
- **Language Support:** Request supported localized API metadata where available
- **No Authentication Required:** The workflow example contains no API key, access token, authorization header, or secret

### Use Cases

- Enrich business or research records with country-level indicators
- Build economic dashboards and development reports
- Compare population, GDP, or other indicators across countries
- Discover indicator codes before constructing data queries
- Support market research, planning, and public-policy analysis

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `resource` | `enum` | Yes | — | Resource to retrieve: `data`, `countries`, `indicators`, `topics`, or `sources` |
| `language` | `string` | No | `en` | API language or locale code, such as `en` |
| `countryCode` | `string` | Conditional | — | Two-letter or supported World Bank country/region code; use `all` for broad queries |
| `indicatorCode` | `string` | Conditional | — | World Bank indicator code, required for `data`, such as `SP.POP.TOTL` |
| `date` | `string` | No | — | Year or range, such as `2024` or `2020:2024`; primarily used by `data` |
| `perPage` | `number` | No | API default | Number of records requested per page for list resources |

### Resource Requirements

| Resource | Required parameters | Example |
|----------|---------------------|---------|
| `data` | `countryCode`, `indicatorCode` | `MA`, `SP.POP.TOTL` |
| `countries` | Optional `countryCode` | `MA` or `all` |
| `indicators` | Optional `countryCode`, `perPage` | `all`, `10` |
| `topics` | Optional `countryCode`, `perPage` | `all`, `10` |
| `sources` | Optional `perPage` | `10` |

### Example Configurations

Population data for Morocco from 2020 through 2024:

```json
{
  "resource": "data",
  "language": "en",
  "countryCode": "MA",
  "indicatorCode": "SP.POP.TOTL",
  "date": "2020:2024"
}
```

List indicators:

```json
{
  "resource": "indicators",
  "language": "en",
  "countryCode": "all",
  "perPage": 10
}
```

### API and Authentication

The World Bank API is publicly accessible and does not require an API key or access token. Requests commonly use the v2 API base URL:

```text
https://api.worldbank.org/v2/
```

JSON output is requested by the node so that results can be consumed by downstream workflow steps.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional dynamic resource value or object containing fields such as `resource`, `countryCode`, `indicatorCode`, `date`, and `perPage`. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | World Bank response for the selected resource, including pagination metadata and resource records when provided |

### Indicator Data Example

World Bank JSON responses commonly contain a metadata object followed by an array of records:

```json
[
  {
    "page": 1,
    "pages": 1,
    "per_page": "50",
    "total": 1
  },
  [
    {
      "indicator": {
        "id": "SP.POP.TOTL",
        "value": "Population, total"
      },
      "country": {
        "id": "MA",
        "value": "Morocco"
      },
      "countryiso3code": "MAR",
      "date": "2024",
      "value": 38158697,
      "unit": "",
      "obs_status": "",
      "decimal": 0
    }
  ]
]
```

### Reference Resource Example

Country, indicator, topic, and source resources return resource-specific records with pagination metadata. Fields differ by resource type.

### Error Output

Invalid country or indicator codes, unsupported languages, malformed date ranges, unavailable data, network failures, and API errors are routed to `error`.

```json
{
  "success": false,
  "error": "World Bank API request failed",
  "resource": "data",
  "countryCode": "MA",
  "indicatorCode": "INVALID"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Retrieve Population Data

```json
{
  "resource": "data",
  "language": "en",
  "countryCode": "MA",
  "indicatorCode": "SP.POP.TOTL",
  "date": "2020:2024"
}
```

### List Country Metadata

```json
{
  "resource": "countries",
  "language": "en",
  "countryCode": "MA"
}
```

### Browse Indicators

```json
{
  "resource": "indicators",
  "language": "en",
  "countryCode": "all",
  "perPage": 10
}
```

### Browse Topics and Sources

```json
{
  "resource": "topics",
  "language": "en",
  "countryCode": "all",
  "perPage": 10
}
```

```json
{
  "resource": "sources",
  "language": "en",
  "perPage": 10
}
```

### Dynamic Configuration

Pass configuration from a previous node through `input`:

```json
{
  "resource": "data",
  "countryCode": "MA",
  "indicatorCode": "NY.GDP.MKTP.CD",
  "date": "2020:2024"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve World Bank development data and reference resources
```

### Common Patterns

- **Indicator lookup:** Manual Trigger → World Bank Data (`data`) → Log
- **Country enrichment:** Customer country → World Bank Data (`countries`) → Update record
- **Indicator discovery:** World Bank Data (`indicators`) → Function → Data query
- **Economic report:** World Bank Data (`data`) → Function → Report or dashboard
- **Public-data catalog:** World Bank Data (`topics`/`sources`) → Storage

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Resource is required

**Cause:** No `resource` was selected.

**Solution:** Choose one of `data`, `countries`, `indicators`, `topics`, or `sources`.

### Indicator data is missing

**Cause:** The `data` resource requires both `countryCode` and `indicatorCode`.

**Solution:** Provide a valid country or region code and an indicator code such as `SP.POP.TOTL` or `NY.GDP.MKTP.CD`.

### No records returned

**Cause:** The indicator has no value for the selected country and period, or the query is too restrictive.

**Solution:** Try `countryCode: "all"`, broaden the date range, or verify the indicator's availability and source.

### Invalid country or indicator code

**Cause:** The value is not recognized by the World Bank API.

**Solution:** Use a two-letter country code or supported aggregate code, and verify indicator codes through the `indicators` resource.

### Unsupported language

**Cause:** The requested language is not supported for the selected resource or field.

**Solution:** Use `language: "en"`, which is the default shown in the workflow examples.

### Pagination or large response

**Cause:** The query returns more records than fit on one page.

**Solution:** Set `perPage`, inspect pagination metadata, and process additional pages where supported.

### Data interpretation concern

**Cause:** Indicators can differ in units, frequency, methodology, revision history, and missing-value conventions.

**Solution:** Read the indicator metadata and source notes before comparing or using values in financial, policy, or operational decisions.

### Authentication question

**Cause:** The workflow appears to require an API credential.

**Solution:** No API key or access token is configured or required for the public World Bank API.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Census.gov](./census-gov.md) — Retrieve public demographic and economic statistics
- [GeoNames](./geo-names.md) — Search geographical places and locations
- [Currency Converter](./currency-converter.md) — Convert monetary values between currencies
- [Log](./log.md) — Inspect World Bank responses

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation |

<!-- /SECTION: changelog -->

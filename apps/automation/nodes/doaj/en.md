---
node_id: "doaj"
title: "DOAJ"
description: "Search Directory of Open Access Journals by ISSN, title or publisher."
category: "search"
subcategory: "specialized"
version: "1.0.0"
language: "en"
last_updated: "2026-11-21"
author: "Fusion Team"
tags:
  - doaj
  - journals
  - open-access
  - research
related_nodes:
  - manual-trigger
  - function
  - http-request
---

<!-- SECTION: overview -->
# DOAJ

> **Type:** Action Node

Search the Directory of Open Access Journals and retrieve journal metadata, identifiers, publisher information and the first available license.

Use an ISSN for a targeted journal lookup, or search titles and publishers to discover matching records. Multiple populated filters are combined with AND. Leaving all search filters blank requests a general search.

A returned record indicates inclusion in DOAJ. The node sets `is_open_access` to true for every returned record. Review the license and source record when assessing a journal's terms.

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `issn` | string | Optional | Empty | ISSN to search, such as `1932-6203`. |
| `title` | string | Optional | Empty | Journal title search, such as `PLOS`. |
| `publisher` | string | Optional | Empty | Publisher name, such as `Public Library of Science (PLoS)`. |
| `page` | number | Optional | `1` | Requested results page. Use a positive integer. |
| `page_size` | number | Optional | `10` | Requested number of records per page. Use a positive integer. |

All fields support Expressions. Surrounding whitespace is removed from search terms when building the API query, while the echoed `query` can retain the original whitespace.

Filters use ISSN, journal title and publisher-name fields in the DOAJ search endpoint. For example, Title PLOS and Publisher Public Library of Science (PLoS) must both match.

The current schema accepts general numbers for pagination. Zero values use the defaults, so Page 0 becomes 1 and Page_size 0 becomes 10. In an observed Fusion run, Page -1 returned Page 1. Other invalid values depend on the API response. Positive integers give predictable requests.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input and precedence

The standard input accepts an object containing `issn`, `title`, `publisher`, `page` and `page_size`.

Upstream object fields are read only when all three search filters in configuration are empty. A configured search filter makes the node use the configured filters and pagination. For dynamic searches, leave ISSN, Title and Publisher blank and connect a Function returning:

```javascript
return {
  title: "PLOS",
  page: 2,
  page_size: 3
};
```

A text string passed as input does not populate a search filter. Use an object or bind a configuration field with the Expression picker. A config field containing spaces is truthy and can prevent the upstream fallback, even though those spaces are removed when constructing the query.

### Outputs

The success output contains:

| Field | Type | Description |
|---|---|---|
| `success` | boolean | True when the node returns a parsed search result. |
| `query` | object | Non-empty search fields echoed from the resolved input. |
| `total_results` | number | Total reported by DOAJ across all matching pages. |
| `page` | number | Page returned by DOAJ, falling back to the resolved request value. |
| `page_size` | number | Page size returned by DOAJ, falling back to the resolved request value. |
| `journals` | array | Records on the current page. Its length can be smaller than the page size. |
| `note` | string | Directory description. |

Each journal includes:

| Fields | Type | Meaning |
|---|---|---|
| `id`, `title`, `alternative_title` | string or null | DOAJ record ID and journal names. A missing title is an empty string. |
| `publisher`, `publisher_country`, `country` | string or null | Publisher name and country mapping. `country` repeats `publisher_country`. |
| `issn`, `eissn` | string arrays | Print/general and electronic identifiers. A journal may have only an eISSN. |
| `language` | array | Reported languages. |
| `subject` | array | Classification entries with `scheme`, `term` and `code`. |
| `keywords` | array | Journal keywords. |
| `start_year`, `end_year` | number or null | Open-access start year, with start-year fallback, and end year when available. |
| `active` | boolean | False only when the upstream active field is explicitly false. |
| `is_open_access` | boolean | Always true for returned directory records. |
| `license`, `license_url` | string or null | First license's type and URL. |
| `journal_url` | string or null | Journal website. |
| `created_date`, `last_updated` | string or null | DOAJ record timestamps. |
| `doaj_url` | string or null | Link constructed from the DOAJ record ID. |

### Observed output

This shortened excerpt comes from the successful ISSN lookup for `1932-6203`. Additional journal fields are described above.

```json
{
  "success": true,
  "query": { "issn": "1932-6203" },
  "total_results": 1,
  "page": 1,
  "page_size": 10,
  "journals": [
    {
      "id": "2fdf1470373343b7bd4f825179c685f5",
      "title": "PLoS ONE",
      "alternative_title": "PLOS ONE",
      "publisher": "Public Library of Science (PLoS)",
      "publisher_country": "US",
      "country": "US",
      "issn": [],
      "eissn": ["1932-6203"],
      "license": "CC BY",
      "license_url": "https://creativecommons.org/licenses/by/4.0/",
      "journal_url": "https://journals.plos.org/plosone/",
      "doaj_url": "https://doaj.org/toc/2fdf1470373343b7bd4f825179c685f5"
    }
  ],
  "note": "DOAJ is a directory of peer-reviewed open access journals"
}
```

A successful search without matches returns `total_results: 0` and `journals: []`. The observed ISSN `0000-0000` query produced this result.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: workflow-example -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Look up an open-access journal in DOAJ
```

The graph preserves the supplied Manual Trigger → DOAJ connection. Configuration values are removed from the exported graph according to the contribution rules.

To reproduce the targeted lookup:

1. Import the example.
2. Open DOAJ and set ISSN to `1932-6203`.
3. Set Title to `PLOS` and Publisher to `Public Library of Science (PLoS)`, as in the supplied workflow.
4. Set Page to `1` and Page_size to `12`.
5. Start Manual Trigger and inspect the DOAJ Output.

The filters are combined, so changing one to an incompatible value can produce an empty result. The supplied graph's Page field was unset; the default is 1.

### Inspect a broader search

Clear ISSN and Publisher, set Title to `PLOS` and Page_size to `3`, then run Page 1 and Page 2. In the observed tests, each page returned three distinct journals and the API reported 14 matches. The total and ordering can change as DOAJ updates its records.

For a compact output, connect a Function after DOAJ:

```javascript
return {
  success: input.success,
  query: input.query,
  total_results: input.total_results,
  page: input.page,
  page_size: input.page_size,
  count: input.journals.length,
  journals: input.journals.map(journal => ({
    id: journal.id,
    title: journal.title,
    publisher: journal.publisher,
    issn: journal.issn,
    eissn: journal.eissn,
    license: journal.license,
    journal_url: journal.journal_url
  }))
};
```

Compare the IDs between pages and inspect the returned page size. `total_results` describes all matches, while `count` in this Function output describes only the current page.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Observation | Explanation and action |
|---|---|
| Empty `journals` | Confirm the spelling and remove filters one at a time. Populated filters must all match. |
| ISSN empty but eISSN populated | The journal may have an electronic identifier only. Inspect both arrays. |
| Upstream fields appear ignored | Clear all three configured search filters, or explicitly bind the desired fields with Expressions. |
| Page_size 0 returns ten journals | Zero uses the default of 10. Use a positive integer for an explicit page size. |
| Negative Page returns Page 1 | This behavior was observed during testing. Use positive integer pagination values. |
| `DOAJ search failed: DOAJ API error: 429` or another HTTP status | The provider returned an unsuccessful status. Check availability and retry later where appropriate. |
| JSON parsing or network error | Inspect connectivity and whether the endpoint returned JSON. The current implementation has no explicit fetch timeout. |
| Optional metadata is missing | The mapping uses null, empty strings or empty arrays for unavailable fields. |
| Success with an unexpectedly empty result | The code defaults a missing `results` field to an empty array. Compare with the API response when diagnosing a malformed response. |

The node returns directory metadata. Its `active` and `is_open_access` values follow the mapping described above and should be interpreted alongside the source record.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
## Related

- [Manual Trigger](/docs/manual-trigger) — Start a journal lookup.
- [Function](/docs/function) — Supply dynamic filters or summarize results.
- [HTTP Request](/docs/http-request) — Inspect an API response.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-11-21 | Document search filters, pagination, input precedence, observed outputs and the sanitized supplied workflow. |

<!-- /SECTION: changelog -->

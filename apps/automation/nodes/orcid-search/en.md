---
node_id: "orcid-search"
title: "ORCID Search"
description: "Search ORCID and retrieve public names, biographies, employments and education records."
category: "search"
subcategory: "specialized"
version: "1.0.0"
language: "en"
last_updated: "2026-11-21"
author: "Fusion Team"
tags:
  - orcid-search
  - research
  - researchers
related_nodes:
  - manual-trigger
  - function
  - http-request
---

<!-- SECTION: overview -->
# ORCID Search

> **Type:** Action Node

Search the ORCID public API, then retrieve the public record for each result. The node returns names, biographies, employment affiliations and education affiliations.

Use a field-qualified ORCID identifier for a targeted lookup, or a broader query to discover candidate profiles. Public profile content depends on what the record exposes. Review identifiers and affiliations before associating a candidate with a person.

The implementation uses `https://pub.orcid.org/v3.0/search/` and fetches `/<orcid-id>/record` for each returned identifier. The node has no API-key configuration field.

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `q` | string | Optional in configuration | Empty | Search expression. A query must resolve from configuration or upstream input at runtime. |
| `start` | number | Optional | `0` | Number of matching records to skip. This is an offset. |
| `rows` | number | Optional | `10` | Requested records per page. The request is capped at 100. |

All fields support Expressions. Use a non-empty query, a non-negative integer Start and a positive integer Rows.

The schema currently accepts general numeric values for pagination. Start 0 is the default; Rows 0 uses 10. Values above 100 are capped in the API request, while the output's `rows` field echoes the resolved configured value. Use Rows from 1 to 100 for consistent request and output limits.

### Query examples

| Q | Purpose |
|---|---|
| `orcid:0000-0002-1825-0097` | Target the Josiah Carberry demonstration profile. |
| `orcid:0000-0002-9322-3515` | Target the Yoshua Bengio profile observed during testing. |
| `Yoshua Bengio` | Free-text search across matching public records. |
| `family-name:Smith` | Search using the family-name field. |

In observed runs, a bare identifier produced a broad result count while the field-qualified `orcid:<id>` query returned one match. Prefer the qualified query when looking up a known identifier.

Free-text search can return people whose names differ from the entered text. In the observed `Yoshua Bengio` run, the first result was Yoshua Bengio and other candidates had different names.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input and precedence

A non-empty configured Q takes priority. Otherwise the node accepts a string or number input, or reads an object using `q`, `query`, then `text`. Start and Rows always come from the node's configuration.

For a dynamic exact lookup, connect a Function returning:

```javascript
return "orcid:0000-0002-1825-0097";
```

Keep Q blank to use the standard input fallback, or select the Function's success value with the Expression picker in Q. The latter path was verified in Fusion.

Object input can use:

```javascript
return { q: "family-name:Smith" };
```

A missing query throws `Search query is required`. The current code checks query presence without trimming whitespace; supply actual search text.

### Output fields

| Field | Type | Meaning |
|---|---|---|
| `success` | boolean | True when the search response is processed. |
| `query` | string | Resolved query sent to the API. |
| `total_results` | number | Total matching records reported by the search API. |
| `start` | number | Resolved offset. |
| `rows` | number | Resolved requested page size. |
| `researchers` | array | Records on this page, excluding results without an ORCID identifier. |
| `note` | string | Description of ORCID identifiers. |

Each researcher contains:

| Field | Type | Meaning |
|---|---|---|
| `orcid_id` | string | ORCID identifier. |
| `orcid_uri` | string when supplied | Identifier URI from the search result. |
| `name` | object | `given`, `family` and `full` strings. |
| `biography` | string or null | Public biography text. |
| `employments` | array | Public employment affiliation summaries. |
| `educations` | array | Public education affiliation summaries. |
| `orcid_url` | string | Public profile URL built from the identifier. |

Employment and education entries contain `organization` with `name`, `city`, `region` and `country`, plus `start_date` and `end_date`. Each date has `year`, `month` and `day` as strings or null. Missing date components remain null.

If an individual record request fails, the node retains its identifier and URL but returns empty name strings, a null biography and empty affiliation arrays. Overall `success` can still be true. An empty array alone cannot distinguish a private or absent affiliation from this detail-fetch fallback.

### Observed output excerpt

This shortened excerpt comes from a successful exact lookup. The complete response also included a biography and two employment entries.

```json
{
  "success": true,
  "query": "orcid:0000-0002-1825-0097",
  "total_results": 1,
  "start": 0,
  "rows": 1,
  "researchers": [
    {
      "orcid_id": "0000-0002-1825-0097",
      "orcid_uri": "https://orcid.org/0000-0002-1825-0097",
      "name": {
        "given": "Josiah",
        "family": "Carberry",
        "full": "Josiah Carberry"
      },
      "educations": [],
      "orcid_url": "https://orcid.org/0000-0002-1825-0097"
    }
  ],
  "note": "ORCID provides persistent digital identifiers for researchers"
}
```

A search with no matches returns `total_results: 0` and `researchers: []`. The observed query `orcid:0000-0000-0000-0000` returned this shape.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: workflow-example -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search ORCID and inspect compact researcher records
```

The example retains one connected path from the supplied export: Manual Trigger → ORCID Search → Function → Log. Configuration values, credentials and variables have been removed according to the contribution rules.

Configure it after importing:

1. Set Q to `Yoshua Bengio`, Start to `0` and Rows to `5`.
2. Paste the following code into the downstream Function.
3. Run Manual Trigger.
4. Inspect the compact results in Log, then inspect ORCID Search's full Output when checking affiliations.

```javascript
return {
  success: input.success,
  query: input.query,
  total_results: input.total_results,
  start: input.start,
  rows: input.rows,
  count: input.researchers.length,
  researchers: input.researchers.map(researcher => ({
    orcid_id: researcher.orcid_id,
    name: researcher.name.full,
    employmentsCount: researcher.employments.length,
    educationsCount: researcher.educations.length,
    orcid_url: researcher.orcid_url
  }))
};
```

The supplied test returned five researchers for the free-text query, including Yoshua Bengio. Counts and result ordering may change.

### Exact lookup

Change Q to `orcid:0000-0002-9322-3515` and Rows to `1`. The observed exact search returned one profile with a biography, one employment and two education entries.

For a stable demonstration query, use `orcid:0000-0002-1825-0097`. This is ORCID's fictional Josiah Carberry demonstration account. Treat its affiliation dates as demonstration data.

### Pagination

Use Q `family-name:Smith` and Rows `2`. Run Start `0` and then Start `2`. The observed runs returned two different identifiers on each page. Compare identifiers across pages and treat `total_results` as the total search count rather than the page length.

### Dynamic query

Add a Function upstream and return the exact query string shown in Inputs. Bind Q to that Function's success output through the Expression picker. Keep Start and Rows configured on ORCID Search.

The tested dynamic path returned the Josiah Carberry profile with `total_results: 1`. Requesting Rows 5 still returns one profile when only one record matches.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Observation | Explanation and action |
|---|---|
| `Search query is required` | Set Q or provide a supported upstream value. Verify that the Expression resolves to search text. |
| A known ID gives a broad result count | Use `orcid:<id>` for a targeted query. |
| Free-text results contain other names | The search returned candidate matches. Review the identifier, public profile and affiliations. |
| Empty `researchers` | The query may have no matches, or the offset may be beyond the available results. |
| Identifier present but name and affiliations empty | A detail request may have failed. Compare the public record and retry when appropriate. |
| Some affiliations or date parts are missing | Public visibility and source completeness vary. Preserve null values and empty arrays. |
| `ORCID search failed: ORCID API error: 429` or another status | The search endpoint returned an HTTP error. Check provider availability and request frequency. |
| Long output is truncated | Use the compact Function above to inspect counts and IDs, then review individual full records. |
| Slow execution | Each result triggers an additional record request, and the implementation has no explicit fetch timeout. Use a smaller Rows value when inspecting results. |

The node exposes public record data. The employment and education mapping does not independently verify the claims in a record.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
## Related

- [Manual Trigger](/docs/manual-trigger) — Start a researcher search.
- [Function](/docs/function) — Supply a query or summarize researcher records.
- [HTTP Request](/docs/http-request) — Inspect a public API response.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0.0 | 2026-11-21 | Document exact and free-text queries, dynamic input, pagination, public-record mapping and the sanitized supplied workflow. |

<!-- /SECTION: changelog -->

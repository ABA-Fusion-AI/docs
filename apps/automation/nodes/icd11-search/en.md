---
node_id: "icd11-search"
title: "ICD11 Search"
description: "Search ICD-11 medical codes and terms through a configurable search endpoint."
category: "healthcare-life-sciences"
subcategory: "icd"
version: "1.0.0"
language: "en"
last_updated: "2026-09-28"
author: "Fusion Team"
tags:
  - healthcare
  - icd11
  - medical-codes
  - search
related_nodes:
  - clinical-tables-search
  - log
---

<!-- SECTION: overview -->
# ICD11 Search

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Search ICD-11 (International Classification of Diseases, 11th Revision) medical codes and terms. The node sends a search request to a configurable endpoint and returns its JSON response unchanged.

Use it to look up terminology, restrict searches to selected chapters or subtrees, or pass search results to another workflow node for processing. The node displays its running status while executing.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

All schema fields are optional. At execution time, a nonblank search query must come from `q` or the input data.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `baseUrl` | `string` | See endpoint below | Search endpoint. An empty string also uses the fallback endpoint. |
| `apiKey` | `string` | `""` | When nonempty, sent in both `Authorization: Bearer <apiKey>` and `X-API-Key`. |
| `q` | `string` | `""` | Search text. A truthy configured value takes precedence over input data. |
| `chapterFilter` | `string` | See chapter list below | Passed as a query parameter when nonempty. Set to `""` to omit it. |
| `subtreesFilter` | `string` | Unset | Passed unchanged as a query parameter when nonempty. |
| `includePostcoordination` | `boolean` | `true` | Requests inclusion of postcoordination information. |
| `useBroaderSynonyms` | `boolean` | `false` | Requests broader synonym matching. |
| `useFlexiSearch` | `boolean` | `false` | Requests flexible search. |
| `includeKeywordResult` | `boolean` | `true` | Requests keyword results. |
| `flatResults` | `boolean` | `true` | Requests flat results. |
| `highlightingEnabled` | `boolean` | `true` | Requests highlighting in results. |
| `medicalCodingMode` | `boolean` | `true` | Requests medical coding mode. |

These flags are forwarded to the endpoint; their detailed effects and the returned fields depend on that service. The node does not implement those search features locally.

### Endpoint

The intended default endpoint is:

```text
https://icd.visiostation.com/icd/release/11/2024-01/mms/search
```

**Source formatting caveat:** The supplied code shows the default and fallback as a Markdown link (`[https://...](https://...)`) inside a string. If copied literally, that value fails `new URL(...)`. Configure `baseUrl` with the plain URL above, or correct both URL literals in the implementation. Examples on this page explicitly use the plain URL.

### Default chapter filter

```text
10;11;12;13;14;15;16;17;18;19;20;21;22;23;24;25;01;02;03;04;05;06;07;08;09;
```

The node forwards this string without parsing or validating chapter identifiers.

### HTTP request

The node uses **POST**, with all search parameters in the URL query string and no request body. It appends parameters to any already present in `baseUrl`; avoid supplying duplicate search parameters there.

| Header | Value |
|--------|-------|
| `Accept` | `application/json` |
| `Content-Type` | `application/json` |
| `accept-language` | `fr` |
| `api-version` | `v2` |
| `Authorization` | `Bearer <apiKey>` when a key is supplied |
| `X-API-Key` | The same key when supplied |

Language and API version are fixed in the implementation. All seven boolean flags are always sent as `"true"` or `"false"` strings. Authentication is omitted when `apiKey` is empty; whether it is required depends on the endpoint.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

| Port | Direction | Description |
|------|-----------|-------------|
| `input` | Input | Input data used as the query when configured `q` is empty. Prefer a string. |
| `success` | Output | The parsed JSON response, returned without transformation. |
| `error` | Output | Workflow error path for failed execution. |

### Query handling

1. Choose configured `q` when truthy. Otherwise, use a string input directly, or convert other input with `String(data || "")`.
2. Reject a missing, empty, or whitespace-only query.
3. Trim surrounding whitespace.
4. Append `%` unless the query already ends with `%`.
5. Encode the query through `URL.searchParams`.

For example, `" asthma "` becomes `"asthma%"`. The node appends the wildcard; its matching behavior is determined by the endpoint.

A whitespace-only configured `q` fails validation even when input contains a valid term. An object such as `{ "q": "asthma" }` is not unpacked; it becomes `"[object Object]%"`. Pass the relevant field as a string instead. Falsy nonstring input such as `0`, `false`, `null`, or `undefined` becomes an empty query and fails validation.

### Response

The node returns the result of `response.json()` directly. It does not enforce a response schema, extract a results array, add metadata, or normalize codes. Inspect an actual response from your endpoint before mapping its fields downstream. An HTTP success response that is not valid JSON still fails execution.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search using a configured term

```json
{
  "baseUrl": "https://icd.visiostation.com/icd/release/11/2024-01/mms/search",
  "q": "asthme"
}
```

The node sends `q=asthme%` (URL-encoded as `q=asthme%25`), the default chapter filter, and the default boolean flags. Input data does not override this configured term.

### Search using input data

```json
{
  "baseUrl": "https://icd.visiostation.com/icd/release/11/2024-01/mms/search",
  "q": "",
  "chapterFilter": "",
  "highlightingEnabled": false
}
```

Pass `"asthme"` to `input`. The query becomes `"asthme%"`, the chapter filter is omitted, and highlighting is disabled in the request.

### Set search options

```json
{
  "baseUrl": "https://icd.visiostation.com/icd/release/11/2024-01/mms/search",
  "q": "asthme%",
  "chapterFilter": "12;",
  "useBroaderSynonyms": true,
  "useFlexiSearch": true,
  "flatResults": false
}
```

The existing trailing `%` is preserved. The chapter filter and flags are forwarded as configured. To restrict a subtree, supply the identifier format accepted by your endpoint in `subtreesFilter`.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: search-example.workflow.json
title: Search ICD-11 and inspect the response
```

Connect **Manual Trigger → ICD11 Search → Log**. Configure ICD11 Search with the plain `baseUrl` and `q: "asthme"` from the first example. Add an API key in your workflow if the service requires one, then run the trigger and inspect the raw JSON in Log. Connect the `error` output to your error-handling branch as needed.

The preview contains the wiring only. Parameters and credentials are omitted; configure the search node before running it.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | Explanation and action |
|------------------|------------------------|
| `Search query (q) is required. Please provide a search term.` | Provide nonblank `q` or string input. Clear whitespace-only `q` to allow input fallback. |
| Invalid URL error | Supply a valid absolute URL. Remove Markdown link syntax from `baseUrl` or the default/fallback literals. URL construction occurs before the request's `try` block, so this error does not receive the `ICD11 search failed:` prefix. |
| `ICD11 search failed: HTTP error! status: <status>, message: <body>` | The endpoint returned a non-2xx status. Review the status and response body; check the endpoint, credentials, and filters. |
| `ICD11 search failed: <message>` | A fetch, response-reading, or JSON parsing error occurred. Check connectivity and whether the endpoint returns valid JSON. |
| Unexpected query text | Objects are stringified rather than searched for a `q` field. Pass the desired field as a string. |
| Unexpected language | The implementation always sends `accept-language: fr`; there is no language setting in this schema. |
| No matching results | Review the query and chapter/subtree restrictions, then inspect the endpoint's returned JSON. Empty results are not treated as an error by the node. |

The implementation defines no automatic retry, explicit timeout, pagination, or response validation. Its `stop()` method is a no-op and does not cancel an in-flight request.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Clinical Tables NLM Search](../clinical-tables-search/en.md)
- Log: inspect the returned JSON in the workflow.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-28 | Initial documentation based on the supplied ICD11SearchNode implementation. |

<!-- /SECTION: changelog -->


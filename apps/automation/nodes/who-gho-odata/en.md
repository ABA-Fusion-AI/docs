---
node_id: "who-gho-odata"
title: "WHO GHO OData"
description: "Access World Health Organization Global Health Observatory data via OData API"
category: "Health / Public Data"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:

- who
- gho
- global-health-observatory
- health-data
- odata
- public-health
- statistics
- api

related_nodes:
- function
- if
- http-request

---

# WHO GHO OData

> **Category:** health-data-nodes | **Type:** Action Node

Access the **World Health Organization's Global Health Observatory (GHO)** data via its **OData** API — dimensions, indicators, indicator data, and arbitrary custom endpoints.

The **WHO GHO OData** node exposes five operations covering GHO's OData model, with rich support for OData query options (`$filter`, `$select`, `$orderby`, `$top`, `$skip`) built from structured, toggleable configuration fields rather than requiring the caller to hand-write raw OData syntax (though raw filter expressions are also supported).

### Supported Features

- List all available dimensions (`getDimensions`)
- List all values for a given dimension (`getDimensionValues`)
- Search/list indicators, with contains- or exact-match name filtering (`getIndicators`)
- Fetch indicator data with structured filters: time range, null-value presence, and custom filter expressions (`getIndicatorData`)
- Arbitrary custom endpoint queries with free-form query parameters (`customQuery`)
- Toggleable OData options: `$select`, `$orderby`, `$top`, `$skip`, applied consistently across `getIndicatorData`, `getIndicators`, and `customQuery`
- OData string escaping for name filters (quote doubling)
- Automatic handling of GHO's underscore-in-URL encoding quirk for indicator codes
- Content-type-aware response parsing (JSON, XML, or plain text)
- Automatic unwrapping of OData's `value` envelope, with `@odata.context`/`@odata.count` metadata surfaced separately

---

## Configuration

### Base Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `baseUrl` | `string` | ❌ No | `"https://ghoapi.azureedge.net/api"` | GHO OData API base URL. |
| `operation` | `enum` | ❌ No | `"getIndicators"` | Operation: `getDimensions`, `getDimensionValues`, `getIndicators`, `getIndicatorData`, or `customQuery`. |

### Operation-Specific Parameters

| Parameter | Type | Required For | Description |
| --------- | ---- | ------------- | ----------- |
| `dimensionCode` | `string` | `getDimensionValues` | Dimension code to list values for (e.g. `"COUNTRY"`, `"REGION"`). |
| `indicatorNameFilter` | `string` | `getIndicators` (optional) | Substring match against indicator names (`contains(...)`). |
| `indicatorNameExact` | `string` | `getIndicators` (optional, ignored if `indicatorNameFilter` is set) | Exact match against indicator names. |
| `indicatorCode` | `string` | `getIndicatorData` | GHO indicator code (e.g. `"WHOSIS_000001"`). |
| `customEndpoint` | `string` | `customQuery` | Raw endpoint path appended to `baseUrl`. |
| `customQueryParams` | `string` (JSON object) | `customQuery` (optional) | Arbitrary query parameters as a JSON object, e.g. `{"$filter": "SpatialDim eq 'MAR'"}`. |

### Filter Options (for `getIndicatorData`)

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `useFilter` | `boolean` | `false` | Enables a raw `filterExpression` filter. |
| `filterExpression` | `string` | — | Raw OData filter expression, used as-is when `useFilter` is `true`. |
| `useTimeFilter` | `boolean` | `false` | Enables time-dimension filtering. |
| `timeYear` | `number` | — | Filters to a single calendar year (1900–2100). Takes priority over `timeStart`/`timeEnd` when set. |
| `timeStart` | `string` | — | Start date (`YYYY-MM-DD`), used if `timeYear` is not set. |
| `timeEnd` | `string` | — | End date (`YYYY-MM-DD`), used if `timeYear` is not set. |
| `useNullFilter` | `boolean` | `false` | Enables filtering by whether a dimension value is null. |
| `nullFilterDimension` | `string` | — | Dimension field to check for null (e.g. `"Dim1"`). |
| `nullFilterType` | `enum` | `"ne"` | `"eq"` filters to null values, `"ne"` filters to non-null values. |

### OData Standard Options (for `getIndicatorData`, `getIndicators`, `customQuery`)

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `useSelect` | `boolean` | `false` | Enables `$select` field limiting. |
| `selectFields` | `string` | — | Comma-separated field names. |
| `useOrderBy` | `boolean` | `false` | Enables `$orderby` sorting. |
| `orderByField` | `string` | — | Field to sort by. |
| `orderByDirection` | `enum` | `"asc"` | `"asc"` or `"desc"`. |
| `useTop` | `boolean` | `false` | Enables `$top` result limiting. |
| `topCount` | `number` | `100` | Max results, 1–10000. |
| `useSkip` | `boolean` | `false` | Enables `$skip` pagination offset. |
| `skipCount` | `number` | `0` | Number of results to skip. |

---

## Operations

| Operation | Endpoint | Description |
| --------- | -------- | ----------- |
| `getDimensions` | `GET /Dimension` | List all GHO dimensions (e.g. country, region, sex, age group). |
| `getDimensionValues` | `GET /DIMENSION/{dimensionCode}/DimensionValues` | List all values for a specific dimension. |
| `getIndicators` | `GET /Indicator` (+ optional `$filter`) | List/search indicators by name. |
| `getIndicatorData` | `GET /{indicatorCode}` (+ built `$filter`, + standard OData options) | Fetch data points for a specific indicator. |
| `customQuery` | `GET /{customEndpoint}` (+ arbitrary params) | Query any GHO OData endpoint not covered above. |

---

## Filter Construction (`getIndicatorData`)

Filters from `useFilter`, `useTimeFilter`, and `useNullFilter` are all **combined with `and`** into a single `$filter` clause, in this order:

1. `filterExpression` (if `useFilter`)
2. Time filter (if `useTimeFilter`) — priority: `timeYear` > (`timeStart` and `timeEnd`) > `timeStart` alone > `timeEnd` alone
3. Null filter (if `useNullFilter` and `nullFilterDimension` is set)

Time filter forms:

| Inputs Set | Resulting Filter Fragment |
| ----------- | -------------------------- |
| `timeYear` | `date(TimeDimensionBegin) ge {year}-01-01 and date(TimeDimensionBegin) lt {year+1}-01-01` |
| `timeStart` + `timeEnd` | `date(TimeDimensionBegin) ge {timeStart} and date(TimeDimensionBegin) lt {timeEnd}` |
| `timeStart` only | `date(TimeDimensionBegin) ge {timeStart}` |
| `timeEnd` only | `date(TimeDimensionBegin) lt {timeEnd}` |

---

## GHO-Specific URL Quirks

- **Indicator codes with underscores**: for `getIndicatorData`, every `_` in `indicatorCode` is replaced with `%5F` before building the URL — GHO's OData service requires this specific encoding rather than accepting a literal underscore or a standard `encodeURIComponent` pass.
- **String escaping**: `indicatorNameFilter`/`indicatorNameExact` values are escaped via `escapeODataString`, which doubles single quotes (`'` → `''`) and double quotes (`"` → `""`) to prevent breaking the generated OData filter syntax.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Always `true` on a successful call. |
| `operation` | `string` | The operation that was performed. |
| `url` | `string` | The final constructed request URL, including query string. |
| `data` | `any` | The response payload — if the raw response had a `value` array (standard OData collection shape), that array is extracted; otherwise the full response body. |
| `metadata` | `object \| undefined` | Present only if the response included `@odata.context`: `{ context, count }` (`count` from `@odata.count`, if present). |
| `count` | `number \| undefined` | Length of `data`, only set if `data` is an array. |
| `statusCode` | `number` | HTTP status code of the response. |

### Response Content-Type Handling

| Content-Type | Handling |
| -------------- | -------- |
| `application/json` | Parsed as JSON (the normal case). |
| `application/xml` / `text/xml` | Returned as `{ xml: "<raw text>", note: "..." }` — **not** parsed into an object. |
| Anything else | Returned as raw text. |

---

## Output Example

### `getIndicators` (filtered)

```json
{
  "success": true,
  "operation": "getIndicators",
  "url": "https://ghoapi.azureedge.net/api/Indicator?$filter=contains(IndicatorName,'life expectancy')",
  "data": [
    {
      "IndicatorCode": "WHOSIS_000001",
      "IndicatorName": "Life expectancy at birth (years)"
    }
  ],
  "metadata": {
    "context": "https://ghoapi.azureedge.net/api/$metadata#Indicator",
    "count": undefined
  },
  "count": 1,
  "statusCode": 200
}
```

### `getIndicatorData` (with time + top)

```json
{
  "success": true,
  "operation": "getIndicatorData",
  "url": "https://ghoapi.azureedge.net/api/WHOSIS%5F000001?$filter=date(TimeDimensionBegin) ge 2023-01-01 and date(TimeDimensionBegin) lt 2024-01-01&$top=5",
  "data": [
    {
      "IndicatorCode": "WHOSIS_000001",
      "SpatialDim": "MAR",
      "TimeDim": 2023,
      "Value": "74.3",
      "TimeDimensionBegin": "2023-01-01T00:00:00Z"
    }
  ],
  "count": 5,
  "statusCode": 200
}
```

---

## Configuration Examples

### List All Dimensions

```json
{
  "operation": "getDimensions"
}
```

### List Country Dimension Values

```json
{
  "operation": "getDimensionValues",
  "dimensionCode": "COUNTRY"
}
```

### Search Indicators by Name

```json
{
  "operation": "getIndicators",
  "indicatorNameFilter": "life expectancy"
}
```

### Indicator Data for a Specific Year, Limited to 10

```json
{
  "operation": "getIndicatorData",
  "indicatorCode": "WHOSIS_000001",
  "useTimeFilter": true,
  "timeYear": 2023,
  "useTop": true,
  "topCount": 10
}
```

### Indicator Data, Non-Null Dim1, Sorted

```json
{
  "operation": "getIndicatorData",
  "indicatorCode": "WHOSIS_000001",
  "useNullFilter": true,
  "nullFilterDimension": "Dim1",
  "nullFilterType": "ne",
  "useOrderBy": true,
  "orderByField": "TimeDim",
  "orderByDirection": "desc"
}
```

### Custom Query

```json
{
  "operation": "customQuery",
  "customEndpoint": "WHOSIS_000001",
  "customQueryParams": "{\"$filter\": \"SpatialDim eq 'MAR'\", \"$top\": \"20\"}"
}
```

---

## Workflow Integration

### Sample Workflow: List Indicators → Function (pick one) → Get Data

```json
{
  "nodes": [
    {
      "id": "who-gho-search",
      "type": "who-gho-odata",
      "config": {
        "operation": "getIndicators",
        "indicatorNameFilter": "maternal mortality"
      }
    },
    {
      "id": "pick-indicator",
      "type": "function"
    },
    {
      "id": "who-gho-data",
      "type": "who-gho-odata",
      "config": {
        "operation": "getIndicatorData"
      }
    }
  ]
}
```

### Sample Workflow: Indicator Data → Chart

```json
{
  "nodes": [
    {
      "id": "who-gho-data",
      "type": "who-gho-odata",
      "config": {
        "operation": "getIndicatorData",
        "indicatorCode": "WHOSIS_000001",
        "useTimeFilter": true,
        "timeStart": "2010-01-01",
        "timeEnd": "2024-01-01"
      }
    },
    {
      "id": "plot-trend",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Schedule → Indicator Data → Database

```json
{
  "nodes": [
    {
      "id": "who-gho-data",
      "type": "who-gho-odata",
      "config": {
        "operation": "getIndicatorData",
        "indicatorCode": "WHOSIS_000001",
        "useTop": true,
        "topCount": 1000
      }
    },
    {
      "id": "store-health-stats",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- WHO GHO OData (`getIndicators`) → Function (extract code) → WHO GHO OData (`getIndicatorData`) — search-then-fetch pipeline
- WHO GHO OData (`getIndicatorData`, time range) → Function → Chart/visualization pipeline — trend analysis
- Schedule → WHO GHO OData → Database — periodic public health data snapshots

---

## Error Handling

### Missing Dimension Code

```text
WHO GHO OData API error: dimensionCode is required for getDimensionValues operation
```

### Missing Indicator Code

```text
WHO GHO OData API error: indicatorCode is required for getIndicatorData operation
```

### Missing Custom Endpoint

```text
WHO GHO OData API error: customEndpoint is required for customQuery operation
```

### Invalid Custom Query Parameters

```text
WHO GHO OData API error: Invalid customQueryParams JSON: <parse error message>
```

Raised when `customQueryParams` is set but is not valid JSON.

### Unknown Operation

```text
WHO GHO OData API error: Unknown operation: <operation>
```

### HTTP Error

```text
WHO GHO OData API error: <response body text, or "HTTP error! status: <status>" if the body is empty/unreadable>
```

Raised when the GHO API returns a non-OK HTTP status — the node attempts to use the raw response body text as the error message before falling back to a generic status message.

All errors are wrapped in the outer `WHO GHO OData API error: ...` message.

---

## Troubleshooting

### "WHO GHO OData API error: dimensionCode is required for getDimensionValues operation"

**Cause**

`dimensionCode` was left empty while `operation` is `getDimensionValues`.

**Solution**

Provide a valid dimension code — run `getDimensions` first to find valid codes (e.g. `"COUNTRY"`, `"REGION"`, `"SEX"`).

---

### "WHO GHO OData API error: indicatorCode is required for getIndicatorData operation"

**Cause**

`indicatorCode` was left empty while `operation` is `getIndicatorData`.

**Solution**

Provide a valid indicator code — run `getIndicators` first (optionally with `indicatorNameFilter`) to find the right code.

---

### "WHO GHO OData API error: Invalid customQueryParams JSON: ..."

**Cause**

`customQueryParams` contains malformed JSON.

**Solution**

Ensure `customQueryParams` is a valid JSON object string, e.g. `{"$filter": "SpatialDim eq 'MAR'"}` — remember to escape inner quotes correctly.

---

### Indicator Data Request Returns Nothing / Wrong Data

**Cause**

Most likely, `indicatorCode` contains an underscore that wasn't recognized correctly, or the constructed `$filter` doesn't match GHO's expected syntax for that indicator's dimension fields (which vary per indicator).

**Solution**

Verify the `url` field in the node's output — it shows the exact constructed request URL — and test it directly in a browser or via `customQuery` with a hand-written filter to isolate the issue.

---

### Time Filter Not Applied As Expected

**Cause**

`timeYear` takes priority over `timeStart`/`timeEnd` when **both** are set — if `timeYear` is populated (even unintentionally, e.g. left over from a previous config), the start/end dates are silently ignored.

**Solution**

Clear `timeYear` if you intend to use `timeStart`/`timeEnd` instead, or vice versa.

---

### XML Response Instead of Expected JSON Data

**Cause**

The GHO API returned an `application/xml`/`text/xml` content type — this node does **not** parse XML into structured data, only wraps the raw text in `{ xml, note }`.

**Solution**

If XML responses are unexpected, verify the request URL and headers; if XML is genuinely expected for a given endpoint, parse `data.xml` downstream with a dedicated XML-parsing `Function` node.

---

### `metadata` is `undefined`

**Cause**

The response did not include an `@odata.context` field — this can happen for non-standard responses (e.g. custom endpoints that don't follow the OData collection shape).

**Solution**

This is expected for some endpoints/operations; check `data` directly rather than relying on `metadata` being present.

---

## Security

The node performs outbound HTTP requests to the configured `baseUrl` (default: the official WHO GHO OData API).

No API key or authentication credential is required — GHO's OData API is fully public.

The `customQuery` operation and `filterExpression` field accept relatively free-form input that is embedded directly into the request URL — while this only affects a GET request to a public health-data API (not a mutation), be mindful that `customEndpoint`/`filterExpression` values sourced from untrusted input could be used to probe unintended parts of the GHO API surface.

---

## Notes

The node's structured filter-building options (`useTimeFilter`, `useNullFilter`, standard OData toggles) are meant to reduce the need to hand-write raw OData syntax, but `useFilter`/`filterExpression` and `customQuery` remain available for cases the structured options don't cover.

The node does not:

- Parse XML responses into structured data (returned as raw text only)
- Validate `indicatorCode`, `dimensionCode`, or filter field names against GHO's actual schema before sending the request
- Cache results between calls
- Support GHO OData's `$count` as a standalone response (only the `@odata.count` metadata field, when present)
- Automatically paginate through all results — `$top`/`$skip` must be used manually across multiple calls

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-27 | Initial release |
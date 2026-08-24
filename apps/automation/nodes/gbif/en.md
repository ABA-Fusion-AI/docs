---
node_id: "gbif-science"
title: "GBIF Science"
description: "Scientific biodiversity data and occurrences."
category: "Biodiversity / Science"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:

- gbif
- biodiversity
- taxonomy
- species
- occurrences
- science
- ecology
- api

related_nodes:
- function
- if
- http-request

---

# GBIF Science

> **Category:** biodiversity-nodes | **Type:** Action Node

Access **biodiversity data** — species taxonomy matching, species details, and occurrence records — from the **Global Biodiversity Information Facility (GBIF)** API.

The **GBIF Science** node exposes three operations: fuzzy species name matching, species detail lookup by taxon key, and occurrence record search by taxon key.

### Supported Features

- Fuzzy species name matching (scientific or common name → best-match taxon)
- Species detail lookup by GBIF taxon key
- Occurrence record search by taxon key, with a configurable result limit
- Direct pass-through of GBIF's raw JSON response (no reshaping)

### Use Cases

- Resolve a common or scientific species name to its canonical GBIF taxon key
- Retrieve taxonomic classification details for a known species
- Retrieve real-world occurrence records (sightings/specimens) for a species
- Build biodiversity research, conservation, or citizen-science workflows
- Combine with an image-identification node (e.g. Pl@ntNet) to enrich a species match with occurrence data

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"search_species"` | Operation to perform: `search_species`, `get_species`, or `get_occurrences`. |
| `scientificName` | `string` | ✅ Yes (for `search_species`) | — | Scientific or common name to match, e.g. `"Panthera leo"`. |
| `taxonKey` | `string` | ✅ Yes (for `get_species`, `get_occurrences`) | — | GBIF taxon key (ID), typically obtained from a prior `search_species` call. |
| `limit` | `number` | ❌ No (for `search_species`, `get_occurrences`) | `20` | Maximum number of results to return. |

Note: `limit` is shown for both `search_species` and `get_occurrences` in the schema, but is only actually used by the `get_occurrences` request — `search_species` calls GBIF's single-match endpoint, which does not accept a limit.

---

## Operations

| Operation | Endpoint | Description |
| --------- | -------- | ----------- |
| `search_species` | `GET /v1/species/match?name={scientificName}` | Fuzzy-matches a name to GBIF's best single taxon match. |
| `get_species` | `GET /v1/species/{taxonKey}` | Gets full taxonomic details for a known taxon key. |
| `get_occurrences` | `GET /v1/occurrence/search?taxonKey={taxonKey}&limit={limit}` | Searches occurrence records for a taxon key. |

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

The node returns the **raw JSON response** from the GBIF API for the selected operation — there is no normalization or field selection.

| Operation | Output |
| --------- | ------ |
| `search_species` | GBIF species-match response: best-matching taxon with `usageKey`, `scientificName`, `canonicalName`, rank, classification (kingdom/phylum/class/order/family/genus/species), and a `confidence` match score. |
| `get_species` | GBIF species-detail response: full taxonomic record for the given `taxonKey`. |
| `get_occurrences` | GBIF occurrence-search response: `{ offset, limit, endOfRecords, count, results }`, where `results` is an array of occurrence records. |

---

## Output Example

### `search_species`

```json
{
  "usageKey": 5219404,
  "scientificName": "Panthera leo (Linnaeus, 1758)",
  "canonicalName": "Panthera leo",
  "rank": "SPECIES",
  "status": "ACCEPTED",
  "confidence": 98,
  "matchType": "EXACT",
  "kingdom": "Animalia",
  "phylum": "Chordata",
  "class": "Mammalia",
  "order": "Carnivora",
  "family": "Felidae",
  "genus": "Panthera",
  "species": "Panthera leo"
}
```

### `get_species`

```json
{
  "key": 5219404,
  "scientificName": "Panthera leo (Linnaeus, 1758)",
  "canonicalName": "Panthera leo",
  "rank": "SPECIES",
  "kingdom": "Animalia",
  "phylum": "Chordata",
  "class": "Mammalia",
  "order": "Carnivora",
  "family": "Felidae",
  "genus": "Panthera",
  "vernacularNames": []
}
```

### `get_occurrences`

```json
{
  "offset": 0,
  "limit": 20,
  "endOfRecords": false,
  "count": 48213,
  "results": [
    {
      "key": 4123456789,
      "scientificName": "Panthera leo (Linnaeus, 1758)",
      "decimalLatitude": -2.153,
      "decimalLongitude": 34.685,
      "country": "Tanzania",
      "eventDate": "2026-03-14",
      "basisOfRecord": "HUMAN_OBSERVATION"
    }
  ]
}
```

---

## Configuration Examples

### Species Name Matching

```json
{
  "operation": "search_species",
  "scientificName": "Panthera leo"
}
```

### Get Species Details

```json
{
  "operation": "get_species",
  "taxonKey": "5219404"
}
```

### Get Occurrences

```json
{
  "operation": "get_occurrences",
  "taxonKey": "5219404",
  "limit": 50
}
```

---

## Workflow Integration

### Sample Workflow: Search → Get Details

```json
{
  "nodes": [
    {
      "id": "gbif-search",
      "type": "gbif-science",
      "config": {
        "operation": "search_species",
        "scientificName": "Panthera leo"
      }
    },
    {
      "id": "gbif-details",
      "type": "gbif-science",
      "config": {
        "operation": "get_species"
      }
    }
  ]
}
```

The second node needs `taxonKey` supplied from the first node's `usageKey` field — typically wired through a `Function` node.

### Sample Workflow: Identify → GBIF Occurrences

```json
{
  "nodes": [
    {
      "id": "plantnet-id",
      "type": "plantnet-id"
    },
    {
      "id": "extract-species-name",
      "type": "function"
    },
    {
      "id": "gbif-search",
      "type": "gbif-science",
      "config": {
        "operation": "search_species"
      }
    },
    {
      "id": "gbif-occurrences",
      "type": "gbif-science",
      "config": {
        "operation": "get_occurrences"
      }
    }
  ]
}
```

### Sample Workflow: Occurrences → Map Visualization

```json
{
  "nodes": [
    {
      "id": "gbif-occurrences",
      "type": "gbif-science",
      "config": {
        "operation": "get_occurrences",
        "taxonKey": "5219404",
        "limit": 100
      }
    },
    {
      "id": "plot-sightings",
      "type": "function"
    }
  ]
}
```

### Common Patterns

- Pl@ntNet ID → Function (extract name) → GBIF (`search_species`) → GBIF (`get_occurrences`) — photo-to-distribution pipeline
- GBIF (`search_species`) → Function (extract `usageKey`) → GBIF (`get_species`) — name-to-taxonomy pipeline
- GBIF (`get_occurrences`) → Function → Map/visualization pipeline — species distribution mapping

---

## Error Handling

### Missing Name

```text
Name is required.
```

Raised for `search_species` when `scientificName` is empty.

### Missing Taxon Key

```text
Taxon Key is required.
```

Raised for `get_species` and `get_occurrences` when `taxonKey` is empty.

The node does not catch or wrap errors from the underlying `fetch`/`.json()` calls — network failures or non-JSON responses will surface as their native JavaScript error, and the node does not check `response.ok` before parsing.

---

## Troubleshooting

### "Name is required."

**Cause**

`scientificName` was left empty while `operation` is `search_species`.

**Solution**

Provide a scientific or common name to match.

---

### "Taxon Key is required."

**Cause**

`taxonKey` was left empty while `operation` is `get_species` or `get_occurrences`.

**Solution**

Provide a valid GBIF taxon key, typically obtained from a prior `search_species` call's `usageKey` field.

---

### `search_species` Returns a Low-Confidence or Unexpected Match

**Cause**

GBIF's fuzzy name matching returns its **single best guess** even for ambiguous, misspelled, or very generic names — there is no way to request alternative candidates through this operation.

**Solution**

Check the `confidence` and `matchType` fields in the response before trusting the result; for ambiguous names, refine the input to be more specific (e.g. include the full binomial name).

---

### Unexpected Error or Crash on Invalid `taxonKey`

**Cause**

The node does not check `response.ok` before calling `.json()` — a non-existent `taxonKey` may result in GBIF returning an error page or empty body that fails to parse as JSON, surfacing as a raw parsing error rather than a clear message.

**Solution**

Verify `taxonKey` is a valid, existing GBIF key (e.g. by first running `search_species`) before calling `get_species` or `get_occurrences`.

---

### `get_occurrences` Returns Fewer Results Than `count`

**Cause**

`count` reflects the total number of matching occurrence records in GBIF, while `results` is limited to `limit` (default 20, max effectively unbounded by this node but subject to GBIF's own API limits).

**Solution**

Use `offset`-based pagination against the GBIF API directly (via an `HTTP Request` node) if more than one page of occurrences is needed — this node does not expose an `offset` parameter.

---

## Security

The node performs outbound HTTP requests to the public GBIF API (`api.gbif.org`).

No API key or authentication credential is required — GBIF's core API is fully public.

---

## Notes

The node returns GBIF's raw JSON response with no reshaping, and does **not** validate the HTTP response status before parsing it as JSON.

The node does not:

- Check `response.ok` before parsing — non-OK responses may surface as JSON parse errors rather than clear HTTP error messages
- Support pagination beyond the `limit` parameter for occurrences (no `offset`)
- Support alternative GBIF endpoints (e.g. `/species/search` for multi-result search, `/occurrence/search` with additional filters like country or date range)
- Cache results between calls
- Validate `taxonKey` format before sending the request

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-24 | Initial release |
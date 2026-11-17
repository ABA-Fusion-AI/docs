---
node_id: "rcsbpdb"
title: "RCSB PDB"
description: "Search, retrieve metadata, and download protein structure files from the RCSB Protein Data Bank."
category: "Bioinformatics"
subcategory: "Protein Structures"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - rcsb
  - pdb
  - protein
  - structure
  - bioinformatics
  - biology
related_nodes:
  - gbif
  - pmc
  - medrxiv-search
  - function
---

<!-- SECTION: overview -->
# RCSB PDB

> **Category:** Bioinformatics &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Search and retrieve protein structures from the **RCSB Protein Data Bank** (`https://www.rcsb.org`), the main public archive of 3D structures of proteins, nucleic acids and complexes. The node supports three operations:
- **`search`:** Find structures by text, by maximum resolution, or with a custom RCSB Search API query.
- **`getMetadata`:** Get the full metadata of one or several PDB entries (title, authors, method, resolution…).
- **`downloadFile`:** Download a structure file in **PDB** or **mmCIF** format.

No API key is required.

### Use Cases

- **Research pipelines:** Search structures on a protein or a disease and collect their metadata for analysis.
- **Structure quality filtering:** Keep only high-resolution structures (e.g. ≤ 2.0 Å) for modeling or docking.
- **File retrieval:** Download structure files to feed molecular viewers, docking tools or storage.
- **Literature & data enrichment:** Combine PDB metadata with publications from PMC or medRxiv.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | `search`, `getMetadata` or `downloadFile`. |
| `searchQuery` | `string` | For `search`* | — | Search term (`text`, `resolution`), or a full RCSB Search API query in JSON (`advanced`). |
| `searchType` | `enum` | No | `text` | `text` (full-text search), `resolution` (filter by resolution) or `advanced` (custom JSON query). |
| `returnType` | `enum` | No | `entry` | Type of results: `entry`, `assembly`, `polymer_entity`, `non_polymer_entity`, `polymer_instance`. |
| `start` | `number` | No | `0` | Pagination offset. |
| `rows` | `number` | No | `20` | Number of results per page. |
| `maxResolution` | `number` | For `resolution` | — | Maximum resolution in Ångström (e.g. `2.0`). |
| `pdbId` | `string` | For `getMetadata` / `downloadFile`* | — | One PDB ID (e.g. `4HHB`). Case-insensitive. |
| `pdbIds` | `string[]` | No | — | Several PDB IDs for `getMetadata` (e.g. `["4HHB", "1CRN"]`). |
| `format` | `enum` | No | `pdb` | File format for `downloadFile`: `pdb` or `mmcif`. |

\* These values can also come from the incoming data (see below).

### Search Types

| `searchType` | What it does | Required |
|--------------|--------------|----------|
| `text` | Full-text search on all entry fields | `searchQuery` |
| `resolution` | Structures with resolution ≤ `maxResolution`, optionally combined with `searchQuery` | `maxResolution` |
| `advanced` | Sends your own RCSB Search API query (JSON). `start`, `rows` and `returnType` are applied to it | `searchQuery` (JSON) or a query object as input |

### Values from the Incoming Data

When the parameters are empty, the node reads the input:

| Operation | Accepted input |
|-----------|----------------|
| `search` | A string used as the search term (or a query object for `advanced`) |
| `getMetadata` | A PDB ID string, an array of IDs, `{ "pdbId": "..." }`, `{ "pdbIds": [...] }`, or a `search` result (`result_set`) |
| `downloadFile` | A PDB ID string, `{ "pdbId": "..." }` or `{ "identifier": "..." }` |

This makes it possible to chain **search → getMetadata** directly: the IDs of the search results are read from `result_set`.

### Authentication

No authentication is needed. The RCSB APIs are public.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` / `object` / `array` | Triggers the node. Can carry the search term or PDB IDs. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Search results, entry metadata or file content (see below). |
| `error` | `Error` | Emitted on missing parameters, unknown IDs or API errors. |

### Output Examples

#### `search`

```json
{
  "query_id": "6c1b6f0e-3a2b-4d7e-9f1a-2b3c4d5e6f70",
  "result_type": "entry",
  "total_count": 1543,
  "result_set": [
    { "identifier": "4HHB", "score": 1.0 },
    { "identifier": "2HHB", "score": 0.97 }
  ]
}
```

#### `getMetadata` (one ID)

Full entry metadata from the RCSB Data API, for example:

```json
{
  "rcsb_id": "4HHB",
  "struct": { "title": "THE CRYSTAL STRUCTURE OF HUMAN DEOXYHAEMOGLOBIN AT 1.74 ANGSTROMS RESOLUTION" },
  "exptl": [{ "method": "X-RAY DIFFRACTION" }],
  "rcsb_entry_info": { "resolution_combined": [1.74], "polymer_entity_count": 2 },
  "rcsb_accession_info": { "initial_release_date": "1984-07-17T00:00:00+0000" }
}
```

#### `getMetadata` (several IDs)

An object keyed by PDB ID. IDs that could not be found are left out.

```json
{
  "4HHB": { "rcsb_id": "4HHB", "...": "..." },
  "1CRN": { "rcsb_id": "1CRN", "...": "..." }
}
```

#### `downloadFile`

```json
{
  "pdbId": "4HHB",
  "format": "pdb",
  "content": "HEADER    OXYGEN TRANSPORT    07-MAR-84   4HHB ...",
  "size": 482355
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search, inspect and download protein structures
```

### How it flows

Each Manual Trigger runs one RCSB PDB node, to try every operation separately:

1. **Text search** for `hemoglobin`.
2. **Advanced search** with a custom JSON query.
3. **Resolution search**: `hemoglobin` structures at ≤ 2.0 Å.
4. **Get metadata** for `4HHB`, then for `4HHB`, `1CRN` and `1BNA`.
5. **Download** `4HHB` in PDB and in mmCIF format.

### Example Configurations

#### Resolution search

```json
{
  "operation": "search",
  "searchType": "resolution",
  "searchQuery": "hemoglobin",
  "maxResolution": 2.0,
  "rows": 5
}
```

#### Advanced search

```json
{
  "operation": "search",
  "searchType": "advanced",
  "searchQuery": "{ \"query\": { \"type\": \"terminal\", \"service\": \"full_text\", \"parameters\": { \"value\": \"hemoglobin\" } }, \"return_type\": \"entry\" }",
  "rows": 5
}
```

#### Download a file

```json
{
  "operation": "downloadFile",
  "pdbId": "4HHB",
  "format": "mmcif"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `RCSB PDB search failed: Search query is required for text search`
- **Cause:** `searchQuery` is empty with `searchType` `text`.
- **Solution:** Fill in `searchQuery`, or send a string from the previous node.

#### Results for `[object Object]`
- **Cause:** `searchQuery` is empty and the input is an object; it is converted to text for `text` and `resolution` searches.
- **Solution:** Set `searchQuery` explicitly, e.g. `{{ input.term }}`.

#### `RCSB PDB search failed: Max resolution is required for resolution search`
- **Cause:** `searchType` is `resolution` without `maxResolution`.
- **Solution:** Set `maxResolution` (e.g. `2.0`).

#### `RCSB PDB search failed: Invalid JSON query format`
- **Cause:** `searchQuery` is not valid JSON with `searchType` `advanced`.
- **Solution:** Validate the JSON; it must follow the RCSB Search API format (`query`, `return_type`…).

#### `RCSB PDB getMetadata failed: PDB entry XXXX not found`
- **Cause:** The PDB ID does not exist.
- **Solution:** Check the ID on `https://www.rcsb.org`. With several IDs, unknown ones are skipped instead.

#### `RCSB PDB downloadFile failed: Failed to download PDB file: 404`
- **Cause:** Unknown ID, or the PDB format is not available for this entry (very large structures exist only in mmCIF).
- **Solution:** Check the ID, or use `format: mmcif`.

#### `RCSB PDB search failed: Unexpected end of JSON input`
- **Cause:** No structure matches the query. RCSB answers with an empty response (HTTP 204), which the node cannot parse.
- **Solution:** Broaden the search term or increase `maxResolution`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Documentation rewritten in the standard format: operations, search types, input resolution, output examples, and example workflow |
| 1.0.0 | 2026-09-04 | Initial documentation |

<!-- /SECTION: changelog -->

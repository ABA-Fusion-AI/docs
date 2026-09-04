---
node_id: rcsb-pdb
title: RCSB PDB
description: Search, retrieve metadata, and download protein structure files from the RCSB Protein Data Bank.
category: Bioinformatics
version: 1.0.0
language: en
last_updated: 2026-09-04
author: ABA Fusion AI
tags:
  - rcsb
  - pdb
  - protein
  - structure
  - bioinformatics
related_nodes: []
---

# RCSB PDB

**Category:** Bioinformatics  
**Type:** Action Node

## Overview

The **RCSB PDB** node allows you to search the RCSB Protein Data Bank, retrieve metadata for PDB entries, and download protein structure files.

It supports:

- Text searches
- Resolution-based searches
- Advanced JSON searches
- Metadata retrieval for a single PDB entry
- Metadata retrieval for multiple PDB entries
- PDB and mmCIF file downloads

## Operations

### Search

Searches the RCSB Protein Data Bank.

Three search types are available:

- `text`
- `resolution`
- `advanced`

#### Text Search

Searches RCSB PDB using a text term such as a protein name.

Example:

```text
hemoglobin
```

#### Resolution Search

Searches structures using a maximum experimental resolution.

A search term can also be provided to combine text and resolution filtering.

Example:

```text
Search Query: hemoglobin
Max Resolution: 2
```

#### Advanced Search

Runs a custom RCSB search query supplied as JSON.

### Get Metadata

Retrieves metadata for one or multiple PDB entries.

For a single entry, provide `pdbId`.

For multiple entries, provide `pdbIds` as an array of PDB IDs.

### Download File

Downloads the structure file associated with a PDB entry.

Supported formats:

- `pdb`
- `mmcif`

## Parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `operation` | enum | No | `search` | Operation to perform: `search`, `getMetadata`, or `downloadFile`. |
| `searchQuery` | string | Conditional | — | Search term or advanced JSON search query. |
| `searchType` | enum | No | `text` | Search type: `text`, `advanced`, or `resolution`. |
| `returnType` | enum | No | `entry` | Type of RCSB result to return. |
| `start` | number | No | `0` | Starting position for paginated search results. |
| `rows` | number | No | `20` | Number of search results to return. |
| `maxResolution` | number | Conditional | — | Maximum resolution for a resolution search. |
| `pdbId` | string | Conditional | — | PDB identifier for metadata retrieval or file download. |
| `pdbIds` | array of strings | Conditional | — | PDB identifiers for retrieving metadata for multiple entries. |
| `format` | enum | No | `pdb` | File format: `pdb` or `mmcif`. |

### Return Types

The `returnType` parameter supports:

```text
entry
assembly
polymer_entity
non_polymer_entity
polymer_instance
```

## Outputs

### Search

Search operations return the matching RCSB search results.

The response can include:

- Result type
- Total number of matches
- Matching PDB identifiers
- Search scores

### Get Metadata

For a single PDB ID, the node returns the metadata associated with that PDB entry.

For multiple PDB IDs, the output contains metadata grouped by PDB ID.

Example:

```json
{
  "4HHB": {
    "...": "metadata"
  },
  "1CRN": {
    "...": "metadata"
  }
}
```

### Download File

The download operation returns:

```json
{
  "pdbId": "4HHB",
  "format": "pdb",
  "content": "...",
  "size": 123456
}
```

| Field | Description |
| --- | --- |
| `pdbId` | PDB identifier. |
| `format` | Downloaded file format. |
| `content` | Structure file content as text. |
| `size` | Size of the returned text content. |

## Examples

### Search by Text

```text
Operation: search
Search Query: hemoglobin
Search Type: text
Return Type: entry
Start: 0
Rows: 5
```

### Search by Resolution

```text
Operation: search
Search Query: hemoglobin
Search Type: resolution
Return Type: entry
Start: 0
Rows: 5
Max Resolution: 2
```

### Advanced Search

```text
Operation: search
Search Type: advanced
Return Type: entry
Start: 0
Rows: 5
```

Example `searchQuery`:

```json
{
  "query": {
    "type": "terminal",
    "service": "full_text",
    "parameters": {
      "value": "hemoglobin"
    }
  },
  "return_type": "entry"
}
```

### Get Metadata for One PDB Entry

```text
Operation: getMetadata
PDB ID: 4HHB
```

### Get Metadata for Multiple PDB Entries

```text
Operation: getMetadata
PDB IDs:
- 4HHB
- 1CRN
- 1BNA
```

Equivalent array:

```json
["4HHB", "1CRN", "1BNA"]
```

### Download a PDB File

```text
Operation: downloadFile
PDB ID: 4HHB
Format: pdb
```

### Download an mmCIF File

```text
Operation: downloadFile
PDB ID: 4HHB
Format: mmcif
```

## cURL Tests

### Text Search

```bash
curl.exe -X POST "https://search.rcsb.org/rcsbsearch/v2/query" ^
  -H "Content-Type: application/json" ^
  --data-raw "{\"query\":{\"type\":\"group\",\"nodes\":[{\"type\":\"terminal\",\"service\":\"full_text\",\"parameters\":{\"value\":\"hemoglobin\"}}],\"logical_operator\":\"and\"},\"return_type\":\"entry\",\"request_options\":{\"paginate\":{\"start\":0,\"rows\":5}}}"
```

### Resolution Search

```bash
curl.exe -X POST "https://search.rcsb.org/rcsbsearch/v2/query" ^
  -H "Content-Type: application/json" ^
  --data-raw "{\"query\":{\"type\":\"group\",\"logical_operator\":\"and\",\"nodes\":[{\"type\":\"terminal\",\"service\":\"full_text\",\"parameters\":{\"value\":\"hemoglobin\"}},{\"type\":\"terminal\",\"service\":\"text\",\"parameters\":{\"attribute\":\"rcsb_entry_info.resolution_combined\",\"operator\":\"less_or_equal\",\"value\":2}}]},\"return_type\":\"entry\",\"request_options\":{\"paginate\":{\"start\":0,\"rows\":5},\"return_all_hits\":false,\"sort\":[{\"sort_by\":\"score\",\"direction\":\"desc\"}]}}"
```

### Get Metadata

```bash
curl.exe -L "https://data.rcsb.org/rest/v1/core/entry/4HHB"
```

### Download PDB File

```bash
curl.exe -L "https://files.rcsb.org/download/4HHB.pdb"
```

### Download mmCIF File

```bash
curl.exe -L "https://files.rcsb.org/download/4HHB.cif"
```

## Troubleshooting

### Search Query Is Required

For a text search, provide a value in `searchQuery`.

Example:

```text
hemoglobin
```

### Max Resolution Is Required

When using the `resolution` search type, provide `maxResolution`.

Example:

```text
2
```

### Invalid Advanced Search Query

For an advanced search, provide a valid RCSB search query in JSON format.

### PDB ID Is Required

For `getMetadata`, provide either a single `pdbId` or an array in `pdbIds`.

For `downloadFile`, provide a single `pdbId`.

### PDB Entry Not Found

Check that the PDB identifier exists and is spelled correctly.

Example of a valid PDB ID:

```text
4HHB
```

### Multiple Metadata Inputs

When retrieving multiple entries, enter the IDs as an array of strings.

Example:

```json
["4HHB", "1CRN", "1BNA"]
```

Do not enter all IDs as one comma-separated string.

### File Download Error

Verify that the PDB ID exists and select one of the supported formats:

```text
pdb
mmcif
```

## Version History

| Version | Date | Changes |
| --- | --- | --- |
| 1.0.0 | 2026-09-04 | Initial release. |

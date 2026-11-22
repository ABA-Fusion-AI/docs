---
node_id: "bio-api"
title: "BioAPI"
description: "Query gene symbols, gene information, expression, Gene Ontology, pathways, drug annotations and functional relations through Multiomix BioAPI."
category: "healthcare-life-sciences"
subcategory: "biology-life-sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-11-22"
author: "Fusion Team"
tags:
  - bioapi
  - genomics
  - genes
  - bioinformatics
related_nodes:
  - manual-trigger
  - function
  - my-gene
  - uni-prot-kb
---

<!-- SECTION: header -->
# BioAPI

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

BioAPI queries the public [Multiomix BioAPI](https://github.com/omics-datascience/BioAPI) for gene names, annotations, expression, Gene Ontology (GO) terms, pathways, pharmacogenomic labels and functional associations. Select one of the 13 operations for each node instance. No API key is configured on this node.

The node returns the provider's parsed response directly. Its shape depends on the operation; there is no shared `success` field.

<!-- /SECTION: header -->

<!-- SECTION: overview -->
## Overview

The implementation uses `https://bioapi.multiomix.org`. It sends GET requests for the finder, single-gene group/drug link, and pathway-gene lookups; the other operations use POST with JSON request bodies.

| Operation | Purpose | Runtime input to provide | Output shape |
|---|---|---|---|
| `geneSymbolsFinder` | Find symbols by text | `query`; optional `limit` | Array of symbol strings |
| `geneSymbols` | Resolve submitted identifiers | `geneIds` | Object keyed by submitted ID, with symbol arrays |
| `informationOfGenes` | Get gene metadata | `geneIds` | Object keyed by known gene symbol |
| `genesOfItsGroup` | Get a gene's group | `geneId` | Group object |
| `expressionOfGenes` | Retrieve expression values | `geneIds`; optional `tissue`, `type` | Object keyed by gene with numeric arrays |
| `genesToTerms` | Find GO terms for genes | `geneIds`; optional GO filters | Array of GO term objects |
| `relatedTerms` | Traverse related GO terms | `termId`; optional depth/relations/root flag | Array of GO term objects |
| `pathwayGenes` | Get genes for a pathway | `pathwaySource` and `pathwayId` | Object with `genes` array |
| `pathwaysInCommon` | Find pathways for genes | `geneIds` | Object with `pathways` array |
| `stringRelations` | Find functional gene associations | `geneId`; optional `minCombinedScore` | Array of relation objects |
| `drugsPharmGkb` | Get PharmGKB drug-label annotations | `geneIds` | Object keyed by gene with annotation arrays |
| `drugsRegulatingGene` | Build a DrugBank search link | `geneId` | Object with `link` URL; not drug records |
| `informationOfOncokb` | Get OncoKB-derived annotations | `geneIds` | Object keyed by known gene |

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

`operation` is the selector, defaulting to `geneSymbolsFinder`. Conditional fields appear for the selected operation. Fill the identifiers or query listed for that operation before running.

| Parameter | Used by | Type / values | Default and guidance |
|---|---|---|---|
| `geneIds` | Seven multi-gene operations in the table above | String containing a JSON array or comma-separated IDs | No default. Enter `["BRCA1","BRCA2"]` and keep at least one gene. |
| `geneId` | `genesOfItsGroup`, `drugsRegulatingGene`, `stringRelations` | String | No default; for example `EGFR` or `TP53`. |
| `query` | `geneSymbolsFinder` | String | No default; for example `BRCA`. |
| `limit` | `geneSymbolsFinder` | Number | Default `50`. Set the maximum number of results to return. |
| `tissue` | `expressionOfGenes` | Select value, such as `Brain` or `Skin` | Optional. Select the exact spelling exposed by the node. |
| `type` | `expressionOfGenes` | `json` or `gzip` | Default `json`. Select the API response format. The node returns parsed data to the workflow. |
| `filterType` | `genesToTerms` | `intersection`, `union`, `enrichment` | If omitted, BioAPI uses `intersection`. `union` includes terms related to either gene; `intersection` requires all genes. |
| `ontologyType` | `genesToTerms` | JSON array string or comma-separated list | For example `["biological_process"]`. Other provider values are `molecular_function` and `cellular_component`. |
| `relationType` | `genesToTerms` | JSON array string or comma-separated list | For example `["involved_in"]`. Provider also supports `enables`, `part_of`, `located_in`; omit this field for `enrichment`. |
| `termId` | `relatedTerms` | GO identifier string | Use digits without the `GO:` prefix, for example `0000079`. |
| `generalDepth` | `relatedTerms` | Number | Optional search depth for non-hierarchical relations; for example `5`. |
| `relations` | `relatedTerms` | JSON array string or comma-separated list | For example `["part_of"]`; provider also supports `regulates` and `has_part`. Blank uses provider defaults. |
| `toRoot` | `relatedTerms` | Numeric flag `0` or `1` | Default `0`. `1` includes ancestors up to a root. |
| `pathwaySource` | `pathwayGenes` | Select one of `kegg`, `biocarta`, `ehmn`, `humancyc`, `inoh`, `netpath`, `pid`, `reactome`, `smpdb`, `signalink`, `wikipathways` | No default. Match the source of the pathway ID. |
| `pathwayId` | `pathwayGenes` | String | No default; for example `hsa04912` with `kegg` or `R-HSA-5633008` with `reactome`. |
| `minCombinedScore` | `stringRelations` | Number | Optional minimum association score; for example `700`. |

For `geneIds`, enter the literal JSON text `["BRCA1","BRCA2"]` in the field; it is a string field in the editor. CSV such as `BRCA1, BRCA2` is also accepted. Use the same list syntax for `ontologyType`, `relationType` and `relations`.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Upstream input

Configured non-empty strings take priority over incoming values. The node also accepts upstream values when the corresponding field is empty:

- A string or number becomes `query` for `geneSymbolsFinder`, `termId` for `relatedTerms`, `geneId` for single-gene operations, and `geneIds` otherwise.
- An array of gene strings or objects can supply gene IDs. Single-gene operations use the first extracted gene. Supported object aliases include `gene`, `geneId`, `gene_id`, `symbol`, `hgnc_symbol` and `ensembl_id`.
- An object can supply `geneIds`/`gene_ids`/`genes`/`symbols`, `query`/`search`/`q`, `termId`/`term_id`/`go_id`, `pathwaySource`/`pathway_source`, `pathwayId`/`pathway_id`, and other matching parameter names.
- Single-key `data`, `json`, `body`, `output`, `result` or `payload` wrappers are unpacked before reading the input.

For example, connect a Function returning `return { query: "BRCA" };` to a `geneSymbolsFinder` node with Query blank. A configured Query takes priority over the Function's query. Use an Expression to map other fields dynamically.

### Returned data

The node does not add a wrapper around successful BioAPI data. These abbreviated examples show the output shapes:

```json
["BRCA1-OT1", "BRCA1P1"]
```

`geneSymbolsFinder` returns an array of gene symbol strings, such as the example above for `query=BRCA` and `limit=2`.

```json
{
  "ENSG00000136244": ["IL6"]
}
```

`geneSymbols` maps each submitted identifier to a list of matching symbols. `informationOfGenes` instead maps known gene names to metadata objects, including names, aliases, genomic positions and external identifiers.

`expressionOfGenes` maps each gene to a numeric expression array, such as `{ "APOE": [893.7, 470.4, 930.6] }` (abbreviated example). The result can contain many values, so use a downstream Function when only counts or a sample are needed.

`genesToTerms` returns an array. Each term can contain `go_id`, `name`, `ontology_type`, `relations_to_genes` and, for `enrichment`, `enrichment_metrics` including `p_value`.

`pathwayGenes` returns `{ "genes": [...] }`. `pathwaysInCommon` returns `{ "pathways": [{ "source": "KEGG", "external_id": "hsa04912", "pathway": "GnRH signaling pathway" }] }` with more entries in a full result. `stringRelations` returns relation objects with `gene_1`, `gene_2`, `combined_score` and evidence fields.

`relatedTerms` returns GO term objects with a `relations` map. `toRoot=1` includes ancestor terms. `drugsRegulatingGene` returns a `link` to DrugBank rather than drug records. `drugsPharmGkb` and `informationOfOncokb` return gene-keyed annotation objects; read their source fields and evidence levels when using these data.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Examples

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Inspect APOE expression output
```

The three-node graph comes from the supplied Fusion export: Manual Trigger → BioAPI → Function. It is reduced to one connected path and omits all exported node parameters, variables and secrets. Configure it after importing:

1. Set BioAPI Operation to `expressionOfGenes`, GeneIds to `["APOE"]`, Tissue to `Brain`, and Type to `json`.
2. In the downstream Function, paste the code below. The Function receives the node result as `input`.
3. Run Manual Trigger. Inspect the Function output for a compact count and the BioAPI output for the full values.

```javascript
const values = input.APOE;
return {
  genes: Object.keys(input),
  count: Array.isArray(values) ? values.length : null,
  allNumeric: Array.isArray(values) &&
    values.every(value => typeof value === "number" && Number.isFinite(value)),
  firstFive: Array.isArray(values) ? values.slice(0, 5) : null
};
```

The Function returns a compact summary of the expression values without copying the full array into the next step.

### Two more configurations

| Goal | Set these fields | Expected shape |
|---|---|---|
| Find a gene symbol | `operation=geneSymbolsFinder`, `query=BRCA`, `limit=2` | Array of up to two strings |
| Check a shared pathway | `operation=pathwaysInCommon`, `geneIds=["EGFR","KRAS"]` | Object with `pathways` array; use an `external_id` and `source` with `pathwayGenes` to inspect one path |

<!-- /SECTION: examples -->

<!-- SECTION: errors -->
## Errors and Empty Results

An error stops the node and appears as `BioAPI <operation> failed: ...`. The message may include an HTTP status and text returned by BioAPI. Use the table to identify the input that led to the error and the next step.

| Message or result | Input that produces it | Meaning and what to enter |
|---|---|---|
| `'query' parameter is mandatory` (HTTP 400) | `geneSymbolsFinder` with Query blank and no upstream query | The API received no search text. Enter a query such as `BRCA`. |
| `'limit' parameter must be a numeric value` (HTTP 400) | `geneSymbolsFinder` with a value such as `-1` or `1.5` | The API rejected the limit. Use a positive whole number such as `10`. |
| `geneId is required for genesOfItsGroup operation` | `genesOfItsGroup` with GeneId blank and no upstream gene | Enter a single gene symbol such as `EGFR`. The same single-gene field is used by `drugsRegulatingGene` and `stringRelations`. |
| `BioAPI Resource not found (404)` | For example, `genesOfItsGroup` with an unknown GeneId | The requested resource was not found. Check the identifier and the selected operation. |
| `pathwaySource and pathwayId are required for pathwayGenes operation` | `pathwayGenes` without one or both fields | Select a source and supply an ID from that source, such as `kegg` with `hsa04912`. |
| `term_id is mandatory` (HTTP 400) | `relatedTerms` with TermId blank | Enter a GO term ID such as `0000079`. |
| `term_id is not on database` (HTTP 400) | `relatedTerms` with an unknown TermId | Check that the term exists and enter the numeric ID without `GO:`. |
| `depth should be a positive integer` (HTTP 400) | `relatedTerms` with GeneralDepth `-1` | Use a non-negative whole number such as `5`. |
| `gene_ids must contain at least one gene symbol` (HTTP 400) | Empty GeneIds with `expressionOfGenes`, `pathwaysInCommon` or `informationOfOncokb` | Enter at least one gene, for example `["APOE"]`. |
| HTTP 500 from `genesToTerms` with empty GeneIds | `genesToTerms` with `[]` or no gene input | The API could not process the empty list. Supply at least one gene, for example `["BRCA1","BRCA2"]`, and run again. |
| `fetch failed` or a timeout message | The service could not be reached or did not answer in time | Check connectivity and service availability, then retry the workflow. |

An empty result is different from an error. `geneSymbolsFinder` can return `[]` when no symbols match; `pathwayGenes` can return `{ "genes": [] }` for an unknown pathway ID; some gene-keyed operations may return `{}` or omit an unknown gene. Check the identifier and operation before treating these as a failed request. Enter JSON arrays exactly, for example `["TNF","IL6"]`; malformed array text may produce an empty result instead of a JSON syntax error.

<!-- /SECTION: errors -->

<!-- SECTION: related -->
## Related Nodes

- [Manual Trigger](/docs/manual-trigger) – Start an example workflow on demand.
- [Function](/docs/function) – Provide dynamic input or summarize large responses.
- [My Gene](/docs/my-gene) – Retrieve gene information from another source.
- [UniProtKB](/docs/uni-prot-kb) – Work with protein information.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-03` | Initial documentation. |
| `1.0.0` | `2026-11-22` | Expanded configuration, output, error interpretation and example workflow guidance. |

<!-- /SECTION: changelog -->

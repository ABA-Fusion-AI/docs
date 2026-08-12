---
node_id: "bar"
title: "BAR API"
description: "Bio-Analytic Resource API for plant genomics data, gene expression, interactions, and more."
category: "Genomics"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:

- bar
- bio-analytic-resource
- genomics
- plant-biology
- gene-expression
- efp
- rna-seq
- gene-annotation
- interactions
- snps
- thalemine
- arabidopsis
- api

related_nodes:
- function
- if
- http-request

---

# BAR API

> **Category:** genomics-nodes | **Type:** Action Node

Access the **Bio-Analytic Resource (BAR)** API from the University of Toronto, a large multi-operation node covering plant genomics data: gene expression, gene annotation, gene-gene interactions, subcellular localizations, sequences, SNPs/structural data, and ThaleMine gene records.

The **BAR API** node exposes a single `operation` selector that maps to one of 45 underlying BAR endpoints. Only the parameters relevant to the selected `operation` are shown, using conditional field visibility.

### Supported Features

- 45 selectable BAR API operations across 13 functional categories
- Configurable `baseUrl` (defaults to the official BAR API)
- Conditional parameter display based on selected `operation`
- Support for both `GET`-style and `POST`-style (JSON payload) operations
- JSON payload parsing for array/object parameters
- Dedicated error handling for malformed JSON payloads
- Unified error wrapping for all API failures

### Use Cases

- Retrieve electronic Fluorescent Pictograph (eFP) gene expression images
- Query gene annotations and gene aliases/isoforms
- Look up gene-gene interaction networks and supporting literature
- Retrieve subcellular localization predictions
- Fetch RNA-Seq and microarray gene expression data
- Retrieve DNA/protein sequences for a gene
- Query SNP, docking, and structural homology data
- Query ThaleMine gene information, GeneRIFs, and publications
- Build plant genomics research and bioinformatics workflows
- Feed genomics results into a `Function` node for downstream analysis

---

## Configuration

### Base Parameter

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `baseUrl` | `string` | ❌ No | `"https://bar.utoronto.ca/api"` | BAR API base URL. |
| `operation` | `enum` | ❌ No | `"getSingleGeneQuery"` | Selects which BAR API operation to perform. See [Operations](#operations) below. |

All other parameters are conditionally shown based on the selected `operation`, grouped by category below.

---

## Operations

### LLaMA

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getLlama` | `llamaGeneId` (string, required) | Get LLaMA data for a gene ID (e.g. `AT3G18850`). |

### eFP Image

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getEfpImageList` | — | List available eFP images. |
| `getEfpDataSource` | `efpSpecies` (string) | Get the eFP data source for a species (e.g. `sorghum`, `arabidopsis`). |
| `getEfpDir` | `efpSpecies` (string) | Get the eFP directory for a species. |
| `getEfpImage` | `efp` (default `efp_arabidopsis`), `view` (default `Developmental_Map`), `mode` (default `Absolute`), `gene1` (default `At1g01010`) | Get an eFP expression image for a single gene. |
| `getEfpImageWithGene2` | `efp`, `view`, `mode`, `gene1`, `gene2` (default `At3g27340`) | Get an eFP expression comparison image for two genes. |

### FastPheno

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getFastPhenoBands` | `fastPhenoSite` (default `pintendre`), `fastPhenoMonth` (default `jan`), `fastPhenoBand` (default `band_1`) | Get FastPheno band data for a site/month/band. |
| `getFastPhenoTrees` | `fastPhenoGenotypeId` (default `C`) | Get FastPheno tree data for a genotype. |

### Gene Annotation

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getGeneAnnotation` | `geneAnnotationQuery` (default `alpha-1 protein`) | Search gene annotations by keyword query. |
| `postGeneAnnotation` | `geneAnnotationPayload` (JSON string, e.g. `{"species":"rice","genes":["LOC_Os01g01010"]}`) | Get gene annotations for a species and gene list. |

### Gene Information

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `postGeneAliases` | `geneInfoSpecies` (default `arabidopsis`), `geneInfoTerms` (JSON array) | Get gene aliases for a list of gene terms. |
| `postGeneIsoforms` | `geneInfoSpecies`, `geneInfoTerms` (JSON array) | Get gene isoforms for a list of gene terms. |
| `getGeneIsoforms` | `geneInfoSpecies`, `geneInfoGeneId` (default `AT1G01020`) | Get isoforms for a single gene. |
| `getGenePublications` | `geneInfoSpecies`, `geneInfoGeneId` | Get publications associated with a gene. |
| `postGeneQuery` | `geneInfoSpecies`, `geneInfoTerms` (JSON array) | Query gene information for a list of gene terms. |
| `getGenesByPosition` | `geneInfoSpecies`, `geneInfoChromosome` (default `1`), `geneInfoStart` (default `3000`), `geneInfoEnd` (default `6000`) | Get genes within a chromosomal position range. |
| `getSingleGeneQuery` | `geneInfoSpecies`, `geneInfoTerm` (default `AT1G01010`) | Query information for a single gene term. |

### Interactions

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `postInteractions` | `interactionsSpecies` (default `rice`), `interactionsGenes` (JSON array) | Get interactions for a list of genes. |
| `getAllTags` | — | List all interaction tags. |
| `getAllGrns` | — | List all gene regulatory networks. |
| `getAllPapers` | — | List all indexed interaction papers. |
| `getPaper` | `interactionsPaperNumber` (default `14`) | Get a paper by its number. |
| `getPaperByAgi` | `interactionsStringAgi` (default `AT2G30530`) | Get papers referencing an AGI identifier. |
| `getPaperByAgiPair` | `interactionsAgi1` (default `AT2G30530`), `interactionsAgi2` (default `AT4G33430`) | Get papers referencing a pair of AGI identifiers. |
| `getInteractionsByRef` | `interactionsPaperId` (default `15`) | Get interactions referenced by a paper ID. |
| `postMFinder` | `interactionsMFinderData` (JSON array of gene pairs) | Run motif-finder analysis on gene pairs. |
| `searchByTag` | `interactionsTag` (default `CBF`) | Search interactions by tag name. |
| `getSingleInteraction` | `interactionsInteractionId` (default `70`) | Get a single interaction by ID. |
| `getInteractions` | `interactionsSpecies`, `interactionsQueryGene` (default `LOC_Os01g52560`) | Get interactions for a query gene. |

### Localizations

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `postLocalizations` | `localizationsSpecies` (default `rice`), `localizationsGenes` (JSON array of gene isoforms) | Get subcellular localizations for a list of gene isoforms. |
| `getLocalizations` | `localizationsSpecies`, `localizationsQueryGene` (default `LOC_Os01g52560.1`) | Get subcellular localization for a single gene isoform. |

### Microarray Gene Expression

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getWorldEfpExpression` | `microarraySpecies` (default `arabidopsis`), `microarrayGeneId` (default `At1g01010`) | Get World eFP microarray expression data for a gene. |

### Proxy

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getAttedApi5` | `proxyGeneId` (default `At1g01010`), `proxyTopN` (number, default `5`) | Proxy request to ATTED-II API v5 for coexpression data. |

### RNA-Seq Gene Expression

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `postRnaSeqExpression` | `rnaSeqPayload` (JSON object, e.g. `{"species":"arabidopsis","database":"single_cell","gene_id":"At1g01010","sample_ids":["cluster0_WT1.ExprMean"]}`) | Get RNA-Seq expression data via a full JSON payload. |
| `getRnaSeqGeneExpression` | `rnaSeqSpecies` (default `arabidopsis`), `rnaSeqDatabase` (default `single_cell`), `rnaSeqGeneId` (default `At1g01010`) | Get RNA-Seq gene expression across all samples. |
| `getRnaSeqGeneExpressionSample` | `rnaSeqSpecies`, `rnaSeqDatabase`, `rnaSeqGeneId`, `rnaSeqSampleId` (default `cluster0_WT1.ExprMean`) | Get RNA-Seq gene expression for a specific sample. |

### Sequence

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getSequence` | `sequenceSpecies` (default `tomato`), `sequenceGeneId` (default `Solyc00g005445.1.1`) | Get the DNA/protein sequence for a gene. |

### SNPs

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getDocking` | `snpsReceptor` (default `bri1`), `snpsLigand` (default `brass`) | Get molecular docking data for a receptor/ligand pair. |
| `getHomologs` | `snpsSearchSpecies` (default `canola`), `snpsSearchGene` (default `BnaA07g31480D`), `snpsTargetSpecies` (default `arabidopsis`) | Get homologous genes across species. |
| `getPhenix` | `snpsFixedPdb` (default `Potri.016G107900.1`), `snpsMovingPdb` (default `AT5G01040.1`) | Get Phenix structural alignment data. |
| `getPymol` | `snpsModel` (default `Potri.016G107900.1`), `snpsSnps` (JSON array, e.g. `["V25L","E26A"]`), `snpsChain` (default `None`) | Get PyMOL structure visualization data with SNP annotations. |
| `getSeqHotspots` | `snpsPval` (default `0.95`), `snpsAraid` (default `AT1G56500.1`), `snpsPopid` (default `Potri.013G007800.2`) | Get sequence-based SNP hotspots. |
| `getStructHotspots` | `snpsPval`, `snpsAraid`, `snpsPopid` | Get structure-based SNP hotspots. |
| `getSampleDefinitions` | `snpsSpecies` (default `tomato`) | Get SNP sample definitions for a species. |
| `getGeneSnps` | `snpsSpecies`, `snpsGeneId` (default `Potri.019G123900.1`) | Get SNPs for a specific gene. |

### ThaleMine

| Operation | Parameters | Description |
| --------- | ---------- | ----------- |
| `getThaleMineGeneInformation` | `thaleMineGeneId` (default `At1g01010`) | Get ThaleMine gene information. |
| `getThaleMineGeneRifs` | `thaleMineGeneId` | Get ThaleMine GeneRIFs (Reference Into Function) for a gene. |
| `getThaleMinePublications` | `thaleMineGeneId` | Get ThaleMine publications associated with a gene. |

---

## Parameters Reference

Full parameter list. Only parameters relevant to the selected `operation` are used at runtime; all others are ignored.

| Parameter | Type | Default | Applies To |
| --------- | ---- | ------- | ---------- |
| `baseUrl` | `string` | `https://bar.utoronto.ca/api` | All operations |
| `operation` | `enum` | `getSingleGeneQuery` | — (selector) |
| `llamaGeneId` | `string` | — | `getLlama` |
| `efpSpecies` | `string` | — | `getEfpDataSource`, `getEfpDir` |
| `efp` | `string` | `efp_arabidopsis` | `getEfpImage`, `getEfpImageWithGene2` |
| `view` | `string` | `Developmental_Map` | `getEfpImage`, `getEfpImageWithGene2` |
| `mode` | `string` | `Absolute` | `getEfpImage`, `getEfpImageWithGene2` |
| `gene1` | `string` | `At1g01010` | `getEfpImage`, `getEfpImageWithGene2` |
| `gene2` | `string` | `At3g27340` | `getEfpImageWithGene2` |
| `fastPhenoSite` | `string` | `pintendre` | `getFastPhenoBands` |
| `fastPhenoMonth` | `string` | `jan` | `getFastPhenoBands` |
| `fastPhenoBand` | `string` | `band_1` | `getFastPhenoBands` |
| `fastPhenoGenotypeId` | `string` | `C` | `getFastPhenoTrees` |
| `geneAnnotationQuery` | `string` | `alpha-1 protein` | `getGeneAnnotation` |
| `geneAnnotationPayload` | `string` (JSON) | — | `postGeneAnnotation` |
| `geneInfoSpecies` | `string` | `arabidopsis` | `postGeneAliases`, `postGeneIsoforms`, `postGeneQuery`, `getGeneIsoforms`, `getGenePublications`, `getGenesByPosition`, `getSingleGeneQuery` |
| `geneInfoTerms` | `string[]` (JSON array) | — | `postGeneAliases`, `postGeneIsoforms`, `postGeneQuery` |
| `geneInfoGeneId` | `string` | `AT1G01020` | `getGeneIsoforms`, `getGenePublications` |
| `geneInfoChromosome` | `string` | `1` | `getGenesByPosition` |
| `geneInfoStart` | `string` | `3000` | `getGenesByPosition` |
| `geneInfoEnd` | `string` | `6000` | `getGenesByPosition` |
| `geneInfoTerm` | `string` | `AT1G01010` | `getSingleGeneQuery` |
| `interactionsSpecies` | `string` | `rice` | `postInteractions`, `getInteractions` |
| `interactionsGenes` | `string` (JSON array) | — | `postInteractions` |
| `interactionsQueryGene` | `string` | `LOC_Os01g52560` | `getInteractions` |
| `interactionsPaperNumber` | `string` | `14` | `getPaper` |
| `interactionsStringAgi` | `string` | `AT2G30530` | `getPaperByAgi` |
| `interactionsAgi1` | `string` | `AT2G30530` | `getPaperByAgiPair` |
| `interactionsAgi2` | `string` | `AT4G33430` | `getPaperByAgiPair` |
| `interactionsPaperId` | `string` | `15` | `getInteractionsByRef` |
| `interactionsMFinderData` | `string` (JSON array) | — | `postMFinder` |
| `interactionsTag` | `string` | `CBF` | `searchByTag` |
| `interactionsInteractionId` | `string` | `70` | `getSingleInteraction` |
| `localizationsSpecies` | `string` | `rice` | `postLocalizations`, `getLocalizations` |
| `localizationsGenes` | `string` (JSON array) | — | `postLocalizations` |
| `localizationsQueryGene` | `string` | `LOC_Os01g52560.1` | `getLocalizations` |
| `microarraySpecies` | `string` | `arabidopsis` | `getWorldEfpExpression` |
| `microarrayGeneId` | `string` | `At1g01010` | `getWorldEfpExpression` |
| `proxyGeneId` | `string` | `At1g01010` | `getAttedApi5` |
| `proxyTopN` | `number` | `5` | `getAttedApi5` |
| `rnaSeqPayload` | `string` (JSON) | — | `postRnaSeqExpression` |
| `rnaSeqSpecies` | `string` | `arabidopsis` | `getRnaSeqGeneExpression`, `getRnaSeqGeneExpressionSample` |
| `rnaSeqDatabase` | `string` | `single_cell` | `getRnaSeqGeneExpression`, `getRnaSeqGeneExpressionSample` |
| `rnaSeqGeneId` | `string` | `At1g01010` | `getRnaSeqGeneExpression`, `getRnaSeqGeneExpressionSample` |
| `rnaSeqSampleId` | `string` | `cluster0_WT1.ExprMean` | `getRnaSeqGeneExpressionSample` |
| `sequenceSpecies` | `string` | `tomato` | `getSequence` |
| `sequenceGeneId` | `string` | `Solyc00g005445.1.1` | `getSequence` |
| `snpsReceptor` | `string` | `bri1` | `getDocking` |
| `snpsLigand` | `string` | `brass` | `getDocking` |
| `snpsSearchSpecies` | `string` | `canola` | `getHomologs` |
| `snpsSearchGene` | `string` | `BnaA07g31480D` | `getHomologs` |
| `snpsTargetSpecies` | `string` | `arabidopsis` | `getHomologs` |
| `snpsFixedPdb` | `string` | `Potri.016G107900.1` | `getPhenix` |
| `snpsMovingPdb` | `string` | `AT5G01040.1` | `getPhenix` |
| `snpsModel` | `string` | `Potri.016G107900.1` | `getPymol` |
| `snpsSnps` | `string` (JSON array) | — | `getPymol` |
| `snpsChain` | `string` | `None` | `getPymol` |
| `snpsPval` | `string` | `0.95` | `getSeqHotspots`, `getStructHotspots` |
| `snpsAraid` | `string` | `AT1G56500.1` | `getSeqHotspots`, `getStructHotspots` |
| `snpsPopid` | `string` | `Potri.013G007800.2` | `getSeqHotspots`, `getStructHotspots` |
| `snpsSpecies` | `string` | `tomato` | `getSampleDefinitions`, `getGeneSnps` |
| `snpsGeneId` | `string` | `Potri.019G123900.1` | `getGeneSnps` |
| `thaleMineGeneId` | `string` | `At1g01010` | `getThaleMineGeneInformation`, `getThaleMineGeneRifs`, `getThaleMinePublications` |

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration, driven by the selected `operation`.

### Outputs

The node returns the raw response from the underlying `BarClient` call for the selected operation. The output shape varies entirely by operation — there is no unified output schema across operations.

---

## Output Example

### `getSingleGeneQuery`

```json
{
  "species": "arabidopsis",
  "term": "AT1G01010",
  "results": [
    {
      "geneId": "AT1G01010",
      "name": "NAC domain containing protein 1",
      "chromosome": "1",
      "start": 3631,
      "end": 5899
    }
  ]
}
```

### `getEfpImage`

```json
{
  "imageUrl": "https://bar.utoronto.ca/efp_arabidopsis/cgi-bin/efpWeb.cgi?...",
  "efp": "efp_arabidopsis",
  "view": "Developmental_Map",
  "mode": "Absolute",
  "gene1": "At1g01010"
}
```

The exact response shape depends entirely on the selected operation and the live BAR API.

---

## Configuration Examples

### Single Gene Query (Default)

```json
{
  "operation": "getSingleGeneQuery",
  "geneInfoSpecies": "arabidopsis",
  "geneInfoTerm": "AT1G01010"
}
```

### eFP Expression Image

```json
{
  "operation": "getEfpImage",
  "efp": "efp_arabidopsis",
  "view": "Developmental_Map",
  "mode": "Absolute",
  "gene1": "At1g01010"
}
```

### Gene Annotation Search (POST)

```json
{
  "operation": "postGeneAnnotation",
  "geneAnnotationPayload": "{\"species\":\"rice\",\"genes\":[\"LOC_Os01g01010\"]}"
}
```

### RNA-Seq Gene Expression

```json
{
  "operation": "getRnaSeqGeneExpression",
  "rnaSeqSpecies": "arabidopsis",
  "rnaSeqDatabase": "single_cell",
  "rnaSeqGeneId": "At1g01010"
}
```

### Gene Interactions

```json
{
  "operation": "getInteractions",
  "interactionsSpecies": "rice",
  "interactionsQueryGene": "LOC_Os01g52560"
}
```

### ThaleMine Gene Information

```json
{
  "operation": "getThaleMineGeneInformation",
  "thaleMineGeneId": "At1g01010"
}
```

---

## Workflow Integration

### Sample Workflow: Gene Query → Function

```json
{
  "nodes": [
    {
      "id": "bar-gene-query",
      "type": "bar-api",
      "config": {
        "operation": "getSingleGeneQuery",
        "geneInfoSpecies": "arabidopsis",
        "geneInfoTerm": "AT1G01010"
      }
    },
    {
      "id": "process-gene-data",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: eFP Image → Notification

```json
{
  "nodes": [
    {
      "id": "bar-efp-image",
      "type": "bar-api",
      "config": {
        "operation": "getEfpImage",
        "efp": "efp_arabidopsis",
        "view": "Developmental_Map",
        "mode": "Absolute",
        "gene1": "At1g01010"
      }
    }
  ]
}
```

### Sample Workflow: Interactions → If → Database

```json
{
  "nodes": [
    {
      "id": "bar-interactions",
      "type": "bar-api",
      "config": {
        "operation": "getInteractions",
        "interactionsSpecies": "rice",
        "interactionsQueryGene": "LOC_Os01g52560"
      }
    },
    {
      "id": "filter-interactions",
      "type": "if"
    },
    {
      "id": "store-interactions",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- BAR API (`getSingleGeneQuery`) → Function → BAR API (`getEfpImage`) — enrich a gene lookup with an expression image
- BAR API (`postGeneQuery`) → Function → Database — batch gene metadata lookups
- BAR API (`getInteractions`) → If → Notification — alert on new interaction data
- BAR API (`getRnaSeqGeneExpression`) → Function → Chart/visualization pipeline

---

## Error Handling

If the selected operation is not recognized, the node throws:

```text
Unknown operation: <operation>
```

If a JSON payload parameter (e.g. `geneAnnotationPayload`, `geneInfoTerms`, `interactionsGenes`) is malformed, the node throws:

```text
BAR API error: Invalid JSON in payload - <parse error message>
```

For all other failures (network errors, non-OK HTTP responses from the underlying `BarClient`), the node throws:

```text
BAR API error: <error message>
```

---

## Troubleshooting

### "Unknown operation: <operation>"

**Cause**

The `operation` value does not match any of the 45 supported operations.

**Solution**

Select a valid `operation` value from the [Operations](#operations) list above.

---

### "BAR API error: Invalid JSON in payload - ..."

**Cause**

A parameter expecting a JSON string (array or object) — such as `geneAnnotationPayload`, `geneInfoTerms`, `interactionsGenes`, `localizationsGenes`, `interactionsMFinderData`, `rnaSeqPayload`, or `snpsSnps` — contains malformed JSON.

**Solution**

Verify the JSON syntax of the affected parameter. For example, `geneInfoTerms` must be a valid JSON array string like `["AT1G01010", "AT1G01020"]`.

---

### "BAR API error: <message>"

**Cause**

The underlying BAR API request failed — this can be a network issue, an invalid gene/species identifier, or a non-OK response from the BAR service.

**Solution**

Verify the required parameters for the selected operation (see the [Parameters Reference](#parameters-reference) table) and confirm the gene ID, species, or identifiers used are valid for the BAR API.

---

### Missing Required Parameter for an Operation

**Cause**

Certain operations require a parameter that has no default (e.g. `llamaGeneId`, `geneAnnotationPayload`, `geneInfoTerms`, `interactionsGenes`) and it was left empty.

**Solution**

Check the [Operations](#operations) tables above to confirm which parameters are required for the selected `operation`, and provide a value.

---

### Unexpected Output Shape

**Cause**

Each of the 45 operations returns a different response shape from the BAR API, and there is no unified output schema.

**Solution**

Use a `Function` node downstream to defensively access the fields relevant to the specific operation used.

---

## Security

The node performs outbound HTTP requests to the configured `baseUrl` (default: the official BAR API at `bar.utoronto.ca`).

No API key or authentication credential is required.

The `baseUrl` parameter can be overridden, allowing requests to a self-hosted or mirrored BAR API instance.

---

## Notes

The node returns the raw response from the underlying BAR API operation rather than a normalized structure.

The node does not:

- Validate gene IDs or species names against a known list
- Cache results between calls
- Normalize output shapes across different operations
- Store or persist genomics data
- Generate visualizations from image or structural data
- Perform its own bioinformatics analysis beyond the wrapped `BarClient` call

It is intended to provide a single, unified entry point to the 45 BAR API operations for downstream workflow processing.

---

## Related

- [Function](./function.md) – Transform and process genomics results per operation
- [If](./if.md) – Route workflows based on query results
- [HTTP Request](./http-request.md) – Make additional custom requests to BAR or related APIs

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-12 | Initial release — 45 operations across 13 categories |
---
node_id: "bar"
title: "BAR API"
description: "Query the Bio-Analytic Resource (BAR) for plant genomics: gene information, expression (eFP, RNA-Seq, microarray), interactions, localizations, sequences, SNPs and ThaleMine data."
category: "Genomics"
subcategory: "Plant Biology"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - bar
  - bio-analytic-resource
  - genomics
  - plant-biology
  - arabidopsis
  - gene-expression
  - interactions
related_nodes:
  - rcsbpdb
  - gbif
  - function
  - http-request
---

<!-- SECTION: overview -->
# BAR API

> **Category:** Genomics &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Access the **Bio-Analytic Resource for Plant Biology** (BAR, University of Toronto — `https://bar.utoronto.ca/api`) from a Fusion workflow. BAR gathers plant genomics data, mainly for *Arabidopsis thaliana*, but also rice, tomato, poplar, sorghum, canola and more. The node exposes **47 operations** in 12 groups:

LLaMA · eFP Image · FastPheno · Gene Annotation · Gene Information · Interactions · Localizations · Microarray Expression · Proxy (ATTED) · RNA-Seq Expression · Sequence · SNPs · ThaleMine

No API key is required. Every parameter has a working default value, so each operation can be tried right away.

### Use Cases

- **Gene lookup:** Get the description, aliases, isoforms and publications of a gene (e.g. `AT1G01010`).
- **Expression analysis:** Retrieve RNA-Seq, single-cell or microarray expression of a gene across tissues and conditions.
- **Interaction networks:** Find protein–protein and protein–DNA interactions, gene regulatory networks and their source papers.
- **Structural biology:** Get homologs, SNP hotspots and protein structure data (PyMOL, Phenix, docking).
- **Visualization:** Generate eFP expression pictures for reports.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Common Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `baseUrl` | `string` | No | `https://bar.utoronto.ca/api` | BAR API base URL. |
| `operation` | `enum` | Yes | `getSingleGeneQuery` | Operation to perform (see tables below). |

Only the parameters of the selected operation are shown in the builder.

### Operations

#### LLaMA, eFP Image & FastPheno

| Operation | Endpoint | Parameters (defaults) |
|-----------|----------|------------------------|
| `getLlama` | `GET /LLaMA/{gene}` | `llamaGeneId` (e.g. `AT3G18850`) |
| `getEfpImageList` | `GET /efp_image/` | — |
| `getEfpDataSource` | `GET /efp_image/get_efp_data_source/{species}` | `efpSpecies` (e.g. `sorghum`) |
| `getEfpDir` | `GET /efp_image/get_efp_dir/{species}` | `efpSpecies` |
| `getEfpImage` | `GET /efp_image/{efp}/{view}/{mode}/{gene1}` | `efp` (`efp_arabidopsis`), `view` (`Developmental_Map`), `mode` (`Absolute`), `gene1` (`At1g01010`) |
| `getEfpImageWithGene2` | `GET /efp_image/{efp}/{view}/{mode}/{gene1}/{gene2}` | same + `gene2` (`At3g27340`) |
| `getFastPhenoBands` | `GET /fastpheno/get_bands/{site}/{month}/{band}` | `fastPhenoSite` (`pintendre`), `fastPhenoMonth` (`jan`), `fastPhenoBand` (`band_1`) |
| `getFastPhenoTrees` | `GET /fastpheno/get_trees/{genotype}` | `fastPhenoGenotypeId` (`C`) |

#### Gene Annotation & Gene Information

| Operation | Endpoint | Parameters (defaults) |
|-----------|----------|------------------------|
| `getGeneAnnotation` | `GET /gene_annotation/{query}` | `geneAnnotationQuery` (`alpha-1 protein`) |
| `postGeneAnnotation` | `POST /gene_annotation/` | `geneAnnotationPayload` — JSON `{"species":"rice","genes":[...]}` |
| `getSingleGeneQuery` | `GET /gene_information/single_gene_query/{species}/{term}` | `geneInfoSpecies` (`arabidopsis`), `geneInfoTerm` (`AT1G01010`) |
| `getGeneIsoforms` | `GET /gene_information/gene_isoforms/{species}/{gene}` | `geneInfoSpecies`, `geneInfoGeneId` (`AT1G01020`) |
| `getGenePublications` | `GET /gene_information/gene_publications/{species}/{gene}` | `geneInfoSpecies`, `geneInfoGeneId` |
| `getGenesByPosition` | `GET /gene_information/genes_by_position/{species}/{chr}/{start}/{end}` | `geneInfoSpecies`, `geneInfoChromosome` (`1`), `geneInfoStart` (`3000`), `geneInfoEnd` (`6000`) |
| `postGeneAliases` | `POST /gene_information/gene_aliases` | `geneInfoSpecies`, `geneInfoTerms` (`["AT1G01010"]`) |
| `postGeneIsoforms` | `POST /gene_information/gene_isoforms/` | `geneInfoSpecies`, `geneInfoTerms` |
| `postGeneQuery` | `POST /gene_information/gene_query` | `geneInfoSpecies`, `geneInfoTerms` |

#### Interactions & Localizations

| Operation | Endpoint | Parameters (defaults) |
|-----------|----------|------------------------|
| `getInteractions` | `GET /interactions/{species}/{gene}` | `interactionsSpecies` (`rice`), `interactionsQueryGene` (`LOC_Os01g52560`) |
| `postInteractions` | `POST /interactions/` | `interactionsSpecies`, `interactionsGenes` — JSON array of genes |
| `getAllTags` / `getAllGrns` / `getAllPapers` | `GET /interactions/all_tags` · `/get_all_grns` · `/get_all_papers` | — |
| `getPaper` | `GET /interactions/get_paper/{n}` | `interactionsPaperNumber` (`14`) |
| `getPaperByAgi` | `GET /interactions/get_paper_by_agi/{agi}` | `interactionsStringAgi` (`AT2G30530`) |
| `getPaperByAgiPair` | `GET /interactions/get_paper_by_agi_pair/{agi1}/{agi2}` | `interactionsAgi1`, `interactionsAgi2` (`AT4G33430`) |
| `getInteractionsByRef` | `GET /interactions/interactions_by_ref/{paperId}` | `interactionsPaperId` (`15`) |
| `searchByTag` | `GET /interactions/search_by_tag/{tag}` | `interactionsTag` (`CBF`) |
| `getSingleInteraction` | `GET /interactions/single_interaction/{id}` | `interactionsInteractionId` (`70`) |
| `postMFinder` | `POST /interactions/mfinder` | `interactionsMFinderData` — JSON gene pairs |
| `getLocalizations` | `GET /loc/{species}/{gene}` | `localizationsSpecies` (`rice`), `localizationsQueryGene` (`LOC_Os01g52560.1`) |
| `postLocalizations` | `POST /loc/` | `localizationsSpecies`, `localizationsGenes` — JSON array of isoforms |

#### Expression, Proxy & Sequence

| Operation | Endpoint | Parameters (defaults) |
|-----------|----------|------------------------|
| `getWorldEfpExpression` | `GET /microarray_gene_expression/world_efp/{species}/{gene}` | `microarraySpecies` (`arabidopsis`), `microarrayGeneId` (`At1g01010`) |
| `getAttedApi5` | `GET /proxy/atted_api5/{gene}/{topN}` | `proxyGeneId` (`At1g01010`), `proxyTopN` (`5`) |
| `getRnaSeqGeneExpression` | `GET /rnaseq_gene_expression/{species}/{db}/{gene}` | `rnaSeqSpecies` (`arabidopsis`), `rnaSeqDatabase` (`single_cell`), `rnaSeqGeneId` (`At1g01010`) |
| `getRnaSeqGeneExpressionSample` | `GET /rnaseq_gene_expression/{species}/{db}/{gene}/{sample}` | same + `rnaSeqSampleId` (`cluster0_WT1.ExprMean`) |
| `postRnaSeqExpression` | `POST /rnaseq_gene_expression/` | `rnaSeqPayload` — full JSON payload |
| `getSequence` | `GET /sequence/{species}/{gene}` | `sequenceSpecies` (`tomato`), `sequenceGeneId` (`Solyc00g005445.1.1`) |

#### SNPs & ThaleMine

| Operation | Endpoint | Parameters (defaults) |
|-----------|----------|------------------------|
| `getDocking` | `GET /snps/docking/{receptor}/{ligand}` | `snpsReceptor` (`bri1`), `snpsLigand` (`brass`) |
| `getHomologs` | `GET /snps/homologs/{species}/{gene}/{target}` | `snpsSearchSpecies` (`canola`), `snpsSearchGene` (`BnaA07g31480D`), `snpsTargetSpecies` (`arabidopsis`) |
| `getPhenix` | `GET /snps/phenix/{fixed}/{moving}` | `snpsFixedPdb`, `snpsMovingPdb` |
| `getPymol` | `GET /snps/pymol/{model}?snps=…&chain=…` | `snpsModel`, `snpsSnps` (JSON array, e.g. `["V25L"]`), `snpsChain` (`None`) |
| `getSeqHotspots` / `getStructHotspots` | `GET /snps/seq_hotspot/…` · `/struct_hotspot/{pval}/{araid}/{popid}` | `snpsPval` (`0.95`), `snpsAraid` (`AT1G56500.1`), `snpsPopid` (`Potri.013G007800.2`) |
| `getSampleDefinitions` | `GET /snps/{species}/samples` | `snpsSpecies` (`tomato`) |
| `getGeneSnps` | `GET /snps/{species}/{gene}` | `snpsSpecies`, `snpsGeneId` (`Potri.019G123900.1`) |
| `getThaleMineGeneInformation` / `getThaleMineGeneRifs` / `getThaleMinePublications` | `GET /thalemine/gene_information/{gene}` · `/gene_rifs/…` · `/publications/…` | `thaleMineGeneId` (`At1g01010`) |

### Authentication

No authentication is needed. The BAR API is public.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Raw JSON response of the BAR API (`null` if the response is empty). |
| `error` | `Error` | Emitted on invalid JSON payloads or API errors. |

### Output Example (`getSingleGeneQuery`)

```json
{
  "wasSuccessful": true,
  "data": {
    "AT1G01010": {
      "gene": "AT1G01010",
      "aliases": ["ANAC001", "NAC001", "NTL10"],
      "brief_description": "NAC domain containing protein 1"
    }
  }
}
```

Most BAR endpoints answer with `{ "wasSuccessful": true, "data": ... }`; the content of `data` depends on the operation.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Try the BAR API operations
```

### How it flows

Each Manual Trigger runs one BAR API node with a different operation (LLaMA, eFP, FastPheno, annotations, gene information, interactions…), so every operation can be tested separately and its output read in a Log node.

### Example Configurations

#### Gene information

```json
{
  "operation": "getSingleGeneQuery",
  "geneInfoSpecies": "arabidopsis",
  "geneInfoTerm": "AT1G01010"
}
```

#### RNA-Seq expression

```json
{
  "operation": "postRnaSeqExpression",
  "rnaSeqPayload": "{\"species\":\"arabidopsis\",\"database\":\"single_cell\",\"gene_id\":\"At1g01010\",\"sample_ids\":[\"cluster0_WT1.ExprMean\"]}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `BAR API error: Invalid JSON in payload - ...`
- **Cause:** A JSON parameter (`interactionsGenes`, `localizationsGenes`, `interactionsMFinderData`, `rnaSeqPayload`, `snpsSnps`) is not valid JSON.
- **Solution:** Use double quotes and valid brackets, e.g. `["LOC_Os01g01080", "LOC_Os01g73310"]`.

#### `Bad Request - Check your parameters. BAR API error (400): ...`
- **Cause:** A species, gene ID or value is not supported by this endpoint (e.g. a rice gene with `arabidopsis`).
- **Solution:** Check that the species and the gene ID format match (`AT1G01010` for Arabidopsis, `LOC_Os…` for rice, `Solyc…` for tomato, `Potri…` for poplar).

#### `Not Found - Resource not found. BAR API error (404)`
- **Cause:** Unknown gene, paper, tag or interaction ID.
- **Solution:** Check the identifier; try the default values first.

#### `BAR API error (400)` on `postGeneAnnotation`, `postGeneAliases`, `postGeneIsoforms`, `postGeneQuery`, `postInteractions`, `postLocalizations` or `postMFinder`
- **Cause:** Known limitation of version 1.0.0: these POST operations do not send the request body expected by BAR (the species is not included, or the payload is sent as text).
- **Solution:** Use the matching GET operation (e.g. `getSingleGeneQuery`, `getInteractions`, `getLocalizations`), or call the endpoint with an **HTTP Request** node. `postRnaSeqExpression` is not affected.

#### `Service Unavailable` / `Internal Server Error`
- **Cause:** The BAR server is busy or under maintenance.
- **Solution:** Retry later.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Documentation rewritten in the standard format: 47 operations with endpoints and defaults, output example, known POST limitations |
| 1.0.0 | 2026-08-12 | Initial documentation |

<!-- /SECTION: changelog -->

---
node_id: "nvidia-bionemo-openfold2-predict"
title: "BioNeMo: OpenFold2 Predict"
description: "Predict protein structures from amino acid sequences using NVIDIA BioNeMo OpenFold2."
category: "Healthcare & Life Sciences"
subcategory: "BioNeMo"
version: "1.0.0"
language: "en"
last_updated: "2026-09-11"
author: "Fusion Team"
tags:
  - healthcare
  - life-sciences
  - bionemo
  - openfold2
  - protein-structure
  - protein-folding
  - bioinformatics
related_nodes:
  - nvidia-bionemo-msa-search
  - nvidia-bionemo-alphafold2-predict
  - nvidia-bionemo-openfold3-predict
  - log
---

<!-- SECTION: header -->

# BioNeMo: OpenFold2 Predict

> **Category:** Healthcare & Life Sciences | **Subcategory:** BioNeMo | **Type:** Action Node

Fast, accurate protein structure prediction using NVIDIA BioNeMo OpenFold2. The node accepts an amino acid sequence and can use multiple sequence alignment (MSA) and template information when supplied through the supported request configuration.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: OpenFold2 Predict** node sends a protein sequence to the NVIDIA BioNeMo OpenFold2 service and returns the predicted structure through the `success` output. Use it to add structure prediction to protein-analysis, bioinformatics, and life-sciences workflows.

The example workflow runs the node with a sample ubiquitin-like protein sequence, selects model `1`, and sends the result to a Log node.

### Key Features

- **Protein structure prediction:** Predict a three-dimensional protein structure from an amino acid sequence
- **OpenFold2 inference:** Use NVIDIA BioNeMo's OpenFold2 prediction service
- **Sequence-based requests:** Provide a protein sequence in standard one-letter amino acid notation
- **MSA and template support:** Enrich prediction requests with supported alignment and template inputs
- **Model selection:** Choose the OpenFold2 model or models used for inference
- **Error routing:** Route authentication, validation, service, and network failures to the `error` output

### Use Cases

- Explore the likely structure of a protein sequence
- Prepare predicted structures for visualization or downstream analysis
- Compare structure predictions across selected models
- Enrich protein annotation and bioinformatics workflows
- Feed MSA- or template-assisted predictions into downstream research steps

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | Yes | — | NVIDIA API key used to authenticate the BioNeMo service |
| `sequence` | `string` | Yes | — | Protein sequence in one-letter amino acid notation |
| `selected_models` | `array<number>` | Yes | `[1]` in the example | OpenFold2 model identifiers selected for prediction |
| MSA inputs | service-supported input | No | — | Optional multiple sequence alignment data for the prediction request |
| Template inputs | service-supported input | No | — | Optional structural template data for the prediction request |

### Example Configuration

```json
{
  "apiKey": "<OPENFOLD2_API_KEY>",
  "sequence": "MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQKESTLHLVLRLRGG",
  "selected_models": [1]
}
```

### Protein Sequence

Provide a valid amino acid sequence using the standard one-letter alphabet. Remove FASTA headers, whitespace, and unsupported characters unless the configured BioNeMo service explicitly accepts them.

### Model Selection

`selected_models` is an array so that one or more supported model identifiers can be selected. The example uses model `1`:

```json
{
  "selected_models": [1]
}
```

Use model identifiers supported by the deployed OpenFold2 service. Selecting more models may increase execution time and response size.

### MSA and Template Inputs

OpenFold2 supports MSA and template-assisted prediction. Supply these through the request fields supported by the current BioNeMo integration when additional evolutionary or structural context is required. Validate the format and identifiers before execution.

### API Key and Secrets

The node requires an NVIDIA API key. Keep the key in protected Fusion secret management or credential configuration rather than committing a real key to a workflow file. Use a placeholder in exported examples:

```text
<OPENFOLD2_API_KEY>
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node. Use it to prepare or provide dynamic prediction request data when supported by the integration. |

The prediction sequence, model selection, API key, and optional MSA or template data are configured on the node or supplied dynamically according to the workflow runtime's input-mapping behavior.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Successful OpenFold2 prediction response, including structure data and service metadata returned by BioNeMo |
| `error` | `object` or `string` | Authentication, validation, service, network, or prediction failure |

The exact structure of the success response depends on the BioNeMo service and Fusion runtime. Connect a Log or downstream analysis node to inspect the returned payload during workflow development.

### Example Success Payload

The service response may include structure coordinates, confidence information, model metadata, or job details. A response shape can vary by deployment, for example:

```json
{
  "structure": "<predicted structure data>",
  "model": 1,
  "sequence": "MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQKESTLHLVLRLRGG"
}
```

Treat the example as illustrative; inspect the actual `success` payload before mapping fields downstream.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Predict a Structure from a Sequence

```json
{
  "apiKey": "<OPENFOLD2_API_KEY>",
  "sequence": "MQIFVKTLTGKTITLEVEPSDTIENVKAKIQDKEGIPPDQQRLIFAGKQLEDGRTLSDYNIQKESTLHLVLRLRGG",
  "selected_models": [1]
}
```

### Select Multiple Models

Use multiple supported model identifiers when the deployed service supports them:

```json
{
  "apiKey": "<OPENFOLD2_API_KEY>",
  "sequence": "MKTIIALSYIFCLVFADYKDDDDK",
  "selected_models": [1, 2]
}
```

### Dynamic Sequence Input

An upstream node can prepare a protein sequence before OpenFold2 runs:

```json
{
  "sequence": "MKTIIALSYIFCLVFADYKDDDDK"
}
```

Keep the API key in protected configuration or secret management, and do not log it with the prediction request.

### MSA- or Template-Assisted Prediction

Connect a sequence-preparation or MSA/template node before OpenFold2 and map the resulting supported fields into the prediction request. Confirm the field names and formats exposed by the current BioNeMo integration before publishing the workflow.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow connects a Manual Trigger to OpenFold2 Predict and sends the successful result to a Log node.

```fusion-workflow
src: example.workflow.json
title: Predict a protein structure with BioNeMo OpenFold2
```

### Common Patterns

- **Basic prediction:** Manual Trigger → BioNeMo: OpenFold2 Predict → Log
- **Dynamic sequence:** Data source → Function or parser → BioNeMo: OpenFold2 Predict
- **MSA-assisted prediction:** BioNeMo: MSA Search → BioNeMo: OpenFold2 Predict
- **Downstream analysis:** OpenFold2 Predict → Structure visualization or analysis node
- **Error handling:** OpenFold2 Predict `error` output → Notification or recovery node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Authentication failed

**Cause:** The NVIDIA API key is missing, invalid, expired, revoked, or unavailable to the BioNeMo service.

**Solution:** Configure a valid key through protected secret management and verify that it has access to the OpenFold2 service.

#### Invalid protein sequence

**Cause:** The sequence is empty, malformed, contains unsupported characters, or includes an unhandled FASTA header.

**Solution:** Validate the sequence, remove unsupported formatting, and retry with a known valid amino acid sequence.

#### Unsupported model selection

**Cause:** `selected_models` contains an identifier that is unavailable in the deployed service.

**Solution:** Use model identifiers supported by the current OpenFold2 deployment and start with the example value `[1]`.

#### MSA or template rejected

**Cause:** The alignment or template data does not match the expected format, identifiers, or sequence.

**Solution:** Validate the auxiliary data and confirm the supported request schema for the configured BioNeMo integration.

#### Prediction timed out or failed

**Cause:** The request is too large, the service is busy, or the BioNeMo endpoint is unavailable.

**Solution:** Start with a shorter valid sequence and one model, verify service availability, and retry. Inspect the `error` output for service details.

#### Node is not registered

**Cause:** The package containing `nvidia-bionemo-openfold2-predict` is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication error | Missing or invalid NVIDIA API key | Configure a valid protected API key |
| Sequence validation error | Invalid or empty amino acid sequence | Validate and normalize the sequence |
| Model validation error | Unsupported model identifier | Select a supported model |
| MSA/template validation error | Auxiliary input is malformed or incompatible | Check the service-supported format |
| Prediction error | OpenFold2 could not complete inference | Review inputs and retry with a smaller request |
| Service or network error | BioNeMo endpoint could not be reached | Check connectivity and service availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [BioNeMo: MSA Search](../nvidia-bionemo-msa-search/en.md) - Prepare or search multiple sequence alignment data
- [BioNeMo: AlphaFold2 Predict](../nvidia-bionemo-alphafold2-predict/en.md) - Run an alternative protein-structure prediction workflow
- [BioNeMo: OpenFold3 Predict](../nvidia-bionemo-openfold3-predict/en.md) - Use the newer OpenFold3 prediction node
- [Log](../log/en.md) - Inspect prediction responses during workflow development

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-11 | Added OpenFold2 protein-structure prediction documentation and workflow guidance |

<!-- /SECTION: changelog -->

---
node_id: "nvidia-bionemo-openfold3-predict"
title: "BioNeMo: OpenFold3 Predict"
description: "Predict biomolecular structures with the latest NVIDIA BioNeMo OpenFold model."
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
  - openfold3
  - structure-prediction
  - protein-folding
  - bioinformatics
related_nodes:
  - nvidia-bionemo-openfold2-predict
  - nvidia-bionemo-msa-search
  - log
---

<!-- SECTION: header -->

# BioNeMo: OpenFold3 Predict

> **Category:** Healthcare & Life Sciences | **Subcategory:** BioNeMo | **Type:** Action Node

Predict biomolecular structures with the latest NVIDIA BioNeMo OpenFold model. The node supports protein, DNA, RNA, and ligand requests, with configurable output format, diffusion sampling, polling, and timeout settings.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: OpenFold3 Predict** node submits a biomolecular sequence or ligand representation to the NVIDIA BioNeMo OpenFold3 service and returns the predicted structure through the `success` output. It can use multiple sequence alignment (MSA) and template inputs when they are provided in the request format supported by the configured service.

The example workflow demonstrates four request types: protein, DNA, RNA, and ligand. Each prediction is connected from a Manual Trigger to the OpenFold3 node and then to a Log node.

### Key Features

- **Latest OpenFold model:** Use the OpenFold3 BioNeMo prediction service
- **Multiple molecule types:** Predict structures for proteins, DNA, RNA, or ligands
- **Configurable output:** Request structures in CIF or PDB format, where supported
- **Diffusion sampling:** Configure the number of diffusion samples used for inference
- **Job polling:** Control polling frequency and the maximum wait time for asynchronous predictions
- **MSA and template support:** Add evolutionary or structural context through supported request fields
- **Error routing:** Route authentication, validation, service, and network failures to the `error` output

### Use Cases

- Predict protein structures from amino acid sequences
- Explore DNA or RNA structural arrangements
- Generate ligand-containing or ligand-focused structure predictions
- Prepare structure files for visualization and downstream analysis
- Build multi-molecule computational biology workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `molecule_type` | `enum` | Yes | `protein` | Molecule type for the request: `protein`, `dna`, `rna`, or `ligand` |
| `output_format` | `enum` | Yes | `cif` | Structure format returned by the service, such as `cif` or `pdb` |
| `apiKey` | `string` | Yes | — | NVIDIA API key used to authenticate the BioNeMo service |
| `sequence` | `string` | Yes | — | Sequence or ligand representation used for the prediction |
| `diffusion_samples` | `number` | No | `1` in the examples | Number of diffusion samples generated for inference |
| `pollSeconds` | `number` | No | `5` in the examples | Number of seconds between status checks for an asynchronous prediction |
| `timeout` | `number` | No | Varies by example | Maximum time to wait for the prediction, in seconds |
| `molecule_id` | `string` | Conditional | — | Identifier used to label a non-protein molecule in the request, such as `A`, `B`, or `L` |
| `molecules` | `array` | No | `[]` in the ligand example | Additional molecule definitions for multi-molecule or ligand-aware requests |

### Example Configuration: Protein

```json
{
  "molecule_type": "protein",
  "output_format": "cif",
  "apiKey": "<OPENFOLD3_API_KEY>",
  "sequence": "MKTIIALSYIFCLVFA",
  "diffusion_samples": 1,
  "pollSeconds": 5,
  "timeout": 1800
}
```

### Molecule Types

Use `molecule_type` to identify the primary input:

- `protein` for amino acid sequences
- `dna` for DNA nucleotide sequences
- `rna` for RNA nucleotide sequences
- `ligand` for a ligand representation supported by the BioNeMo service

The sequence alphabet and any additional request fields must match the selected molecule type. For mixed or complex predictions, use the supported `molecule_id` and `molecules` fields.

### Output Format

The examples use `cif` for protein, DNA, and RNA requests, and `pdb` for the ligand request. Choose a format supported by the deployed service and downstream tools:

```json
{
  "output_format": "cif"
}
```

### Diffusion and Job Controls

- `diffusion_samples` controls the number of samples generated during inference. Larger values may provide more candidate structures but can increase execution time and output size.
- `pollSeconds` controls the interval between asynchronous job-status checks.
- `timeout` sets the maximum wait period in seconds. Set it high enough for the molecule size and selected sampling configuration.

The example uses `diffusion_samples: 1` and polls every five seconds. Its timeout ranges from 1,200 to 2,000 seconds depending on the request type.

### MSA and Template Inputs

OpenFold3 supports MSA and template-assisted prediction. Supply these through the request fields supported by the current BioNeMo integration when additional evolutionary or structural context is required. Validate their format and correspondence with the input sequence before execution.

### API Key and Secrets

The node requires an NVIDIA API key. Keep the key in protected Fusion secret management or credential configuration rather than committing a real key to a workflow file. Use a placeholder in exported examples:

```text
<OPENFOLD3_API_KEY>
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node. It can provide dynamic prediction request data when supported by the integration. |

The node configuration defines the molecule type, sequence or ligand representation, output format, API key, sampling controls, and optional molecule metadata. Upstream nodes can prepare dynamic values before execution.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Successful OpenFold3 response, including predicted structure data and service metadata |
| `error` | `object` or `string` | Authentication, validation, timeout, service, network, or prediction failure |

The exact success response depends on the BioNeMo service and Fusion runtime. The structure content may be returned as CIF or PDB data, according to `output_format`. Connect a Log or downstream analysis node to inspect the actual payload.

### Illustrative Success Payload

```json
{
  "structure": "<predicted CIF or PDB data>",
  "molecule_type": "protein",
  "output_format": "cif"
}
```

Treat this payload as illustrative; response field names and structure data packaging can vary by deployment.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Protein Prediction

```json
{
  "molecule_type": "protein",
  "output_format": "cif",
  "apiKey": "<OPENFOLD3_API_KEY>",
  "sequence": "MKTIIALSYIFCLVFA",
  "diffusion_samples": 1,
  "pollSeconds": 5,
  "timeout": 1800
}
```

### DNA Prediction

```json
{
  "molecule_type": "dna",
  "output_format": "cif",
  "apiKey": "<OPENFOLD3_API_KEY>",
  "sequence": "ATGCGTACGTTAGC",
  "diffusion_samples": 1,
  "pollSeconds": 5,
  "timeout": 1200,
  "molecule_id": "B"
}
```

### RNA Prediction

```json
{
  "molecule_type": "rna",
  "output_format": "cif",
  "apiKey": "<OPENFOLD3_API_KEY>",
  "sequence": "AUGCGUACGU",
  "diffusion_samples": 1,
  "pollSeconds": 5,
  "timeout": 1800,
  "molecule_id": "A"
}
```

### Ligand Prediction

```json
{
  "molecule_type": "ligand",
  "output_format": "pdb",
  "apiKey": "<OPENFOLD3_API_KEY>",
  "sequence": "CCO",
  "diffusion_samples": 1,
  "pollSeconds": 5,
  "timeout": 2000,
  "molecule_id": "L",
  "molecules": []
}
```

### MSA- or Template-Assisted Prediction

Connect an MSA or template preparation node before OpenFold3 and map the resulting supported fields into the prediction request. Confirm the field names and formats exposed by the current BioNeMo integration before publishing the workflow.

### Dynamic Request Values

An upstream node can prepare a sequence and molecule metadata before execution:

```json
{
  "molecule_type": "protein",
  "sequence": "MKTIIALSYIFCLVFA",
  "output_format": "cif"
}
```

Keep the API key in protected configuration or secret management, and do not send it to logging nodes.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow contains four Manual Trigger → OpenFold3 Predict → Log paths demonstrating protein, DNA, RNA, and ligand requests.

```fusion-workflow
src: example.workflow.json
title: Predict biomolecular structures with BioNeMo OpenFold3
```

### Common Patterns

- **Protein prediction:** Manual Trigger → BioNeMo: OpenFold3 Predict → Log
- **Nucleic-acid prediction:** Sequence source → BioNeMo: OpenFold3 Predict → Structure analysis
- **Ligand-aware prediction:** Molecule preparation → BioNeMo: OpenFold3 Predict → Visualization or scoring
- **MSA-assisted prediction:** BioNeMo: MSA Search → BioNeMo: OpenFold3 Predict
- **Error handling:** OpenFold3 Predict `error` output → Notification or recovery node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Authentication failed

**Cause:** The NVIDIA API key is missing, invalid, expired, revoked, or unavailable to the BioNeMo service.

**Solution:** Configure a valid key through protected secret management and verify access to the OpenFold3 service.

#### Invalid sequence or ligand input

**Cause:** The input is empty, malformed, uses the wrong alphabet, or is incompatible with `molecule_type`.

**Solution:** Validate the input for the selected molecule type and retry with a known valid example.

#### Unsupported output format

**Cause:** The selected `output_format` is unavailable for the deployed service or molecule type.

**Solution:** Try `cif` or `pdb` as appropriate and confirm the formats supported by the deployment.

#### Molecule metadata is invalid

**Cause:** `molecule_id` or `molecules` does not match the selected molecule type or request schema.

**Solution:** Use the expected identifier and validate additional molecule definitions before submission.

#### Prediction timed out

**Cause:** The request exceeded `timeout`, possibly because of sequence size, multiple samples, or service load.

**Solution:** Increase the timeout when appropriate, start with one diffusion sample, and confirm the service is available.

#### MSA or template rejected

**Cause:** Auxiliary alignment or template data is malformed, incompatible, or does not correspond to the input molecule.

**Solution:** Validate the auxiliary data and confirm the request schema supported by the configured BioNeMo integration.

#### Node is not registered

**Cause:** The package containing `nvidia-bionemo-openfold3-predict` is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication error | Missing or invalid NVIDIA API key | Configure a valid protected API key |
| Input validation error | Invalid sequence, ligand, or molecule type | Validate and normalize the request |
| Format validation error | Unsupported CIF or PDB selection | Choose a supported output format |
| Molecule metadata error | Invalid ID or additional molecule definition | Check `molecule_id` and `molecules` |
| Timeout error | Prediction exceeded the configured wait period | Increase `timeout` or reduce request complexity |
| Prediction error | OpenFold3 could not complete inference | Review inputs and retry with one sample |
| Service or network error | BioNeMo endpoint could not be reached | Check connectivity and service availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [BioNeMo: OpenFold2 Predict](../nvidia-bionemo-openfold2-predict/en.md) - Predict protein structures with OpenFold2
- [BioNeMo: MSA Search](../nvidia-bionemo-msa-search/en.md) - Prepare or search multiple sequence alignment data
- [Log](../log/en.md) - Inspect prediction responses during workflow development

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-11 | Added OpenFold3 biomolecular-structure prediction documentation and workflow guidance |

<!-- /SECTION: changelog -->

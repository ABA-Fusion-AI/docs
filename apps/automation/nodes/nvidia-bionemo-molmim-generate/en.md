---
node_id: "nvidia-bionemo-molmim-generate"
title: "BioNeMo: MolMIM Generate"
description: "Generate molecules optimized for target properties using MolMIM"
category: "Healthcare & Life Sciences"
subcategory: "BioNeMo"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - healthcare
  - life-sciences
  - bionemo
  - molmim
  - molecular-generation
  - drug-discovery
  - optimization
related_nodes:
  - nvidia-bionemo-filter-molecules
  - nvidia-bionemo-genmol-generate
  - log
---

<!-- SECTION: header -->

# BioNeMo: MolMIM Generate

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Generate molecules optimized for a target property such as QED or penalized logP (plogP) using NVIDIA BioNeMo MolMIM.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: MolMIM Generate** node performs property-optimized molecular generation. Starting from a seed molecule represented as a SMILES string, MolMIM uses an optimization algorithm to generate new molecules that aim to improve a selected molecular property.

The example workflow uses the CMA-ES algorithm, optimizes QED, generates three molecules, and sends the result to a Log node.

### Key Features

- **Property optimization:** Optimize generated molecules for a selected molecular property
- **Seed-based generation:** Start generation from a supplied SMILES string
- **CMA-ES optimization:** Use the Covariance Matrix Adaptation Evolution Strategy in the example configuration
- **Configurable search:** Control the number of generated molecules, iterations, and particles
- **Workflow-ready outputs:** Route generated molecules or errors to downstream nodes
- **NVIDIA authentication:** Authenticate requests with an NVIDIA API key

### Use Cases

- Explore molecular candidates with improved drug-likeness
- Optimize compounds for QED or penalized logP
- Generate candidate molecules from a known chemical scaffold
- Support early-stage drug-discovery workflows
- Feed generated molecules into filtering, docking, or property-analysis nodes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `algorithm` | `enum` | ✅ Yes | `CMA-ES` in the example | Optimization algorithm used for molecular generation |
| `property_name` | `enum` or `string` | ✅ Yes | `QED` in the example | Molecular property to optimize |
| `apiKey` | `string` | ✅ Yes | — | NVIDIA API key used to authenticate the BioNeMo service |
| `smi` | `string` | ✅ Yes | — | Seed molecule in SMILES notation |
| `num_molecules` | `number` | ✅ Yes | `3` in the example | Number of molecules to generate |
| `iterations` | `number` | ✅ Yes | `5` in the example | Number of optimization iterations |
| `particles` | `number` | ✅ Yes | `10` in the example | Number of candidate particles evaluated during optimization |

### Example Configuration

```json
{
  "algorithm": "CMA-ES",
  "property_name": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smi": "CC(=O)OC1=CC=CC=C1C(=O)O",
  "num_molecules": 3,
  "iterations": 5,
  "particles": 10
}
```

### Optimization Algorithm

The example uses:

```text
CMA-ES
```

CMA-ES searches for candidate molecules that improve the selected property relative to the seed structure. The available algorithms depend on the BioNeMo service configuration.

### Target Property

The example optimizes:

```text
QED
```

The `property_name` parameter selects the molecular property used by the optimization process. The example description also identifies plogP as a supported target property. Use a property supported by the configured MolMIM service.

### Generation Controls

- `num_molecules` controls how many candidate molecules are requested.
- `iterations` controls how many optimization cycles are performed.
- `particles` controls the number of candidate solutions considered during the search.

Higher values may increase exploration and execution time.

### API Key and Secrets

The node requires an NVIDIA API key. The example workflow currently contains a hard-coded `apiKey` value in its node parameters.

Use a protected Fusion credential or secret reference instead of placing a real key directly in workflow JSON. The documentation and exported examples should contain only a placeholder such as:

```text
<NVIDIA_API_KEY>
```

**Security notice:** Rotate or revoke the exposed key in `apps/automation/nodes/nvidia-bionemo-molmim-generate/example.workflow.json` before sharing or committing the workflow.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node |

The node configuration defines the seed SMILES, optimization property, algorithm, API key, and generation controls. An upstream node can be used to prepare dynamic molecular input before execution.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Successful MolMIM generation result containing generated molecule data |
| `error` | `object` or `string` | Authentication, validation, service, network, or generation failure |

The exact success response depends on the BioNeMo MolMIM service response and the Fusion runtime. Use a Log node to inspect the returned structure during workflow development.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Optimize QED with CMA-ES

Generate three molecules from an aspirin-like seed structure and optimize for QED.

```json
{
  "algorithm": "CMA-ES",
  "property_name": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smi": "CC(=O)OC1=CC=CC=C1C(=O)O",
  "num_molecules": 3,
  "iterations": 5,
  "particles": 10
}
```

### Optimize Penalized LogP

Use plogP as the target property when supported by the configured MolMIM service.

```json
{
  "algorithm": "CMA-ES",
  "property_name": "plogP",
  "apiKey": "<NVIDIA_API_KEY>",
  "smi": "CCO",
  "num_molecules": 3,
  "iterations": 5,
  "particles": 10
}
```

### Dynamic Seed Molecule

An upstream node can provide a seed SMILES value:

```json
{
  "smi": "C1=CC=CC=C1"
}
```

Keep the API key and optimization settings in protected node configuration or secret management.

### Example Error Case

A missing or invalid API key can prevent the request from being authenticated:

```json
{
  "algorithm": "CMA-ES",
  "property_name": "QED",
  "apiKey": "<NVIDIA_API_KEY>",
  "smi": "CCO",
  "num_molecules": 3,
  "iterations": 5,
  "particles": 10
}
```

Inspect the `error` output for the service response and authentication details. Do not log the real API key.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow connects a Manual Trigger to the MolMIM node and sends the successful result to a Log node.

```fusion-workflow
src: example.workflow.json
title: Generate property-optimized molecules with MolMIM
```

### Common Patterns

- **Property optimization:** Manual Trigger → BioNeMo: MolMIM Generate → Log
- **Dynamic generation:** Data source → Function → BioNeMo: MolMIM Generate
- **Candidate filtering:** MolMIM Generate → BioNeMo: Filter Molecules
- **Structure analysis:** MolMIM Generate → Molecular property or docking node
- **Error handling:** MolMIM Generate `error` output → Notification or recovery node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Authentication failed

**Cause:** The NVIDIA API key is missing, invalid, expired, revoked, or not available to the configured service.

**Solution:** Configure a valid NVIDIA API key through protected secret management. Rotate the key if it has been exposed in a workflow file.

#### Invalid SMILES input

**Cause:** The seed molecule is malformed or is not accepted by the MolMIM service.

**Solution:** Validate the SMILES string before execution and test with a known valid molecule.

#### Unsupported property

**Cause:** The selected `property_name` is not supported by the configured MolMIM service.

**Solution:** Use a supported property such as `QED` or `plogP`, depending on service capabilities.

#### Generation failed

**Cause:** The optimization request could not generate candidates or was rejected by the service.

**Solution:** Check the seed SMILES, algorithm, property name, and generation controls. Reduce `iterations` or `particles` when troubleshooting resource-related failures.

#### Request timed out

**Cause:** A large number of molecules, iterations, or particles increased execution time.

**Solution:** Start with small values, such as three molecules, five iterations, and ten particles, then increase them gradually.

#### Node is not registered

**Cause:** The package containing the `nvidia-bionemo-molmim-generate` node is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication error | Missing or invalid NVIDIA API key | Configure or rotate the protected API key |
| Invalid SMILES | Malformed seed molecule | Validate the SMILES input |
| Unsupported property | Target property is unavailable | Select a supported property |
| Validation error | Missing or invalid generation parameter | Check algorithm and numeric controls |
| Generation error | MolMIM could not produce candidates | Adjust inputs and optimization settings |
| Service or network error | BioNeMo service could not be reached | Check connectivity and service availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [BioNeMo: Filter Molecules](../nvidia-bionemo-filter-molecules/en.md) - Filter or select generated molecular candidates
- [BioNeMo: GenMol Generate](../nvidia-bionemo-genmol-generate/en.md) - Generate molecules with another BioNeMo workflow
- [Log](./log.md) - Inspect MolMIM generation results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Added MolMIM molecular-generation documentation and security guidance |

<!-- /SECTION: changelog -->


---
node_id: "nvidia-bionemo-boltz2-predict"
title: "BioNeMo: Boltz2 Predict"
description: "Predict structures using Boltz2 with polymers, ligands, alignments, modifications, and bonds."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-25"
author: "Fusion Team"
tags: [integration, peer-only,nvidia, bionemo, boltz2, structure-prediction]
related_nodes: []
---

<!-- SECTION: overview -->
# BioNeMo: Boltz2 Predict

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Prepare a Boltz2 structure-prediction request containing proteins, DNA, RNA, or ligands. The node accepts optional multiple sequence alignments (MSAs), polymer modifications, and bond definitions, and selects mmCIF as the structure output format.

### Use Cases

- Submit a protein sequence for structure prediction.
- Describe complexes using multiple polymer chains and ligands.
- Include alignment data, cyclic polymers, or residue modifications.
- Configure recycling, sampling, and the number of diffusion samples.

> **Implementation issue:** In the supplied source, `calculate_affinity` is referenced without being declared or destructured from `this.config`. This causes a TypeScript compilation error, or a `ReferenceError` before the request if executed without type checking. The handler must be corrected before the examples can run. This document describes its schema and intended request construction; it does not claim a successful prediction from the supplied implementation.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

The schema includes shared fields through `bionemoBaseShape`. The supplied example workflow uses `apiKey`, but the base schema and `BaseBioNeMoNode` implementation were not provided. Their required fields, defaults, expression support, authentication behavior, timeouts, and retry handling cannot be established from this node alone.

### Prediction Parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `polymers` | Array of polymer objects | Conditional | None | Polymer entities. At least one polymer or ligand is required by the handler. |
| `ligands` | Array of ligand objects | Conditional | None | Ligand entities. At least one polymer or ligand is required by the handler. |
| `bonds` | Array of bond objects | No | None | Bond definitions forwarded when the array is non-empty. |
| `recycling_steps` | Integer | No | `3` | Allowed range: 1–10. |
| `sampling_steps` | Integer | No | `50` | Allowed range: 10–200. |
| `diffusion_samples` | Integer | No | `1` | Allowed range: 1–5. |
| `step_scale` | Number | No | `1.638` | Allowed range: 0.5–3.0. |
| `output_format` | Enum | No | `mmcif` | Only `mmcif` is accepted. |
| `calculate_affinity` | Boolean | No | `false` | Declared in the schema, but currently referenced incorrectly in the handler. |

These node-specific fields have no explicit expression metadata in the supplied schema. Arrays and nested objects are structured values, not JSON-encoded strings.

### Polymer Objects

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `id` | String | Yes | None | Polymer identifier. |
| `molecule_type` | Enum | No | `protein` | `protein`, `dna`, or `rna`. |
| `sequence` | String | Yes | None | Polymer sequence. |
| `msa` | Nested record | No | None | Two levels of string keys leading to alignment records. |
| `cyclic` | Boolean | No | `false` | Whether the polymer is cyclic. |
| `modifications` | Array of objects | No | None | At most three modifications per polymer. Each requires `ccd` (string) and `position` (integer). |

The node schema does not constrain identifier length, sequence alphabet, sequence length, or modification position range. It also does not establish a residue-indexing convention.

Each MSA leaf record requires `alignment` (string) and accepts `format` (string, default `a3m`). The format is not restricted to an enum. The nested structure is:

```json
{
  "msa": {
    "database_name": {
      "a3m": {
        "format": "a3m",
        "alignment": ">query\nMKTVRQERLK\n"
      }
    }
  }
}
```

### Ligand Objects

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | String | Yes | Ligand identifier. |
| `ccd` | String | No | Chemical Component Dictionary code. |
| `smiles` | String | No | SMILES representation. |

The local schema permits neither, either, or both of `ccd` and `smiles`; it contains no rule requiring exactly one. Server-side requirements may be stricter.

### Bond Objects

Every bond requires all six fields:

| Field | Type | Description |
| --- | --- | --- |
| `entity1_type` | String | First endpoint's entity type value. |
| `entity1_residue` | Integer | First endpoint's residue value. |
| `entity1_atom` | String | First endpoint's atom value. |
| `entity2_type` | String | Second endpoint's entity type value. |
| `entity2_residue` | Integer | Second endpoint's residue value. |
| `entity2_atom` | String | Second endpoint's atom value. |

The node forwards bond objects unchanged. It does not validate endpoint references, permitted type or atom names, or residue ranges.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operation

The intended action calls:

```text
makeJsonRequest("biology/mit/boltz2/predict", {
  method: "POST",
  body: data
})
```

The shared base class is responsible for executing the request.

### Request Construction

1. Reject input when both `polymers` and `ligands` are missing or empty.
2. Add the five numeric/output settings, using the documented defaults when appropriate.
3. Add `polymers`, `ligands`, and `bonds` only when their arrays are non-empty.
4. Intend to add `calculate_affinity` only when it is truthy.
5. Pass the body to `makeJsonRequest`.

Step 4 currently fails because `calculate_affinity` is not declared in the handler. Setting the configuration field to `false` does not avoid evaluation of the undeclared variable. The direct code correction is to include `calculate_affinity` in the destructuring of `this.config`.

The handler also destructures `without_potentials` and `concatenate_msas` but never includes them in the request body. Neither is declared in the node-specific schema shown here; their availability through the shared schema is unknown.

### API Compatibility

NVIDIA documents the hosted endpoint as `https://health.api.nvidia.com/v1/biology/mit/boltz2/predict`. Its current reference requires 1–12 polymers and lists `constraints`, while this node allows ligand-only input and forwards `bonds`. The reference also lists affinity-specific controls rather than the node's `calculate_affinity` field. Check deployed API compatibility before relying on those options; acceptance by this local schema does not guarantee acceptance by the service. See the [NVIDIA Boltz2 API reference](https://docs.api.nvidia.com/nim/reference/mit-boltz2-infer).
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** A workflow event triggers the action. The handler ignores the incoming payload and reads prediction values from configuration.
- **Success, after correcting the handler:** The value returned by `makeJsonRequest` is returned unchanged. The node performs no additional extraction, conversion, or file writing.
- **Output format:** `mmcif` is a request option. This handler does not itself save an mmCIF file.
- **Failure:** Local validation and request-helper errors propagate. The current undeclared-variable issue prevents valid requests from reaching the helper.

The base helper and response schema are not included in the supplied source, so exact response keys, asynchronous job handling, and error formatting are not specified here.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: errors -->
## Errors & Troubleshooting

| Error or condition | Explanation / action |
| --- | --- |
| `At least one polymer or ligand must be provided for prediction` | Provide a non-empty `polymers` or `ligands` array. Bonds alone do not satisfy the check. |
| Cannot find name `calculate_affinity` / `calculate_affinity is not defined` | Add `calculate_affinity` to the handler's configuration destructuring before execution. |
| Schema validation failure | Check numeric ranges, integer fields, molecule types, output format, required nested fields, and the three-modification limit. |
| Ligand-only request rejected by the API | The local handler allows it, but the current hosted API reference requires polymers. |
| Bond or affinity option rejected / ineffective | Check the request fields against the deployed API version. The supplied node and current reference differ. |
| `without_potentials` or `concatenate_msas` has no effect | The supplied handler reads these values but does not forward them. |
| Authentication, network, or service failure | Check shared BioNeMo connection configuration and the request helper's reported error. Exact behavior depends on the base implementation. |
<!-- /SECTION: errors -->

<!-- SECTION: examples -->
## Examples

These are configuration examples, not verified prediction results. Correct the handler issue first and supply the shared connection settings required by your runtime. The `apiKey` field follows the existing example workflow.

### Single Protein

```json
{
  "apiKey": "<nvidia-api-key>",
  "polymers": [
    {
      "id": "A",
      "molecule_type": "protein",
      "sequence": "MKTVRQERLK"
    }
  ],
  "recycling_steps": 3,
  "sampling_steps": 50,
  "diffusion_samples": 1,
  "step_scale": 1.638,
  "output_format": "mmcif",
  "calculate_affinity": false
}
```

### Polymer with a Ligand

```json
{
  "apiKey": "<nvidia-api-key>",
  "polymers": [
    {
      "id": "A",
      "molecule_type": "protein",
      "sequence": "MKTVRQERLK",
      "cyclic": false
    }
  ],
  "ligands": [
    {
      "id": "B",
      "smiles": "CCO"
    }
  ],
  "diffusion_samples": 2,
  "output_format": "mmcif"
}
```

The short sequences and ligand above illustrate configuration structure only.

### Example Workflow

The included workflow connects a Manual Trigger to BioNeMo: Boltz2 Predict and a Log node. Replace the API-key placeholder before running it. Execution remains subject to the handler correction and API compatibility checks described above.

```fusion-workflow
src: example.workflow.json
title: Use BioNeMo Boltz2 Predict in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Keep real API keys out of documentation and shared workflow exports. Prediction inputs are passed to the configured service through the shared BioNeMo request helper; use an endpoint appropriate for the molecular data you submit.
<!-- /SECTION: security -->


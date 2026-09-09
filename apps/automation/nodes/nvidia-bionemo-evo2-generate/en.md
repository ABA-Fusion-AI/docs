---
node_id: "nvidia-bionemo-evo2-generate"
title: "BioNeMo: Evo2 Generate"
description: "Generate DNA and RNA sequences using the NVIDIA BioNeMo Evo2 foundation model"
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
  - evo2
  - dna
  - rna
  - sequence-generation
  - genomics
related_nodes:
  - nvidia-bionemo-msa-search
  - nvidia-bionemo-smiles-to-sdf
  - log
---

<!-- SECTION: header -->

# BioNeMo: Evo2 Generate

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Generate DNA or RNA sequences with the NVIDIA BioNeMo Evo2 foundation model, trained on 9.3 trillion nucleotides.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioNeMo: Evo2 Generate** node generates nucleotide sequences from a supplied DNA or RNA seed sequence. It supports configurable sampling controls, deterministic random seeds, optional token probabilities, elapsed-time metrics, and logits output.

The example workflow uses the seed sequence `ATGCGTACGTTAGCTA`, generates 10 tokens, and sends the successful result to a Log node.

### Key Features

- **DNA and RNA generation:** Generate nucleotide sequences from a seed sequence
- **Foundation model:** Use the BioNeMo Evo2 model
- **Sampling controls:** Configure temperature, top-k, and top-p sampling
- **Reproducibility:** Set a random seed for repeatable generation
- **Optional diagnostics:** Include sampled probabilities, elapsed time per token, and logits
- **Polling controls:** Configure polling frequency and request timeout
- **NVIDIA authentication:** Authenticate requests with an NVIDIA API key

### Use Cases

- Explore possible DNA or RNA sequence continuations
- Generate candidate sequences for genomics research
- Prototype sequence design workflows
- Analyze token probabilities and generation metrics
- Pass generated sequences to downstream bioinformatics or validation nodes

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | NVIDIA API key used to authenticate the BioNeMo service |
| `sequence` | `string` | ✅ Yes | — | DNA or RNA seed sequence used as the generation prompt |
| `num_tokens` | `number` | ✅ Yes | `10` in the example | Number of new tokens to generate |
| `temperature` | `number` | No | `1` in the example | Sampling temperature controlling generation diversity |
| `top_k` | `number` | No | `4` in the example | Limits sampling to the top K candidate tokens |
| `top_p` | `number` | No | `0.9` in the example | Nucleus-sampling probability threshold |
| `random_seed` | `number` | No | `42` in the example | Seed used to make generation reproducible |
| `enable_sampled_probs` | `boolean` | No | `true` in the example | Include sampled-token probabilities in the response |
| `enable_elapsed_ms_per_token` | `boolean` | No | `true` in the example | Include elapsed generation time per token |
| `pollSeconds` | `number` | No | `2` in the example | Number of seconds between status polls |
| `timeout` | `number` | No | `900` in the example | Maximum time allowed for the generation request |
| `enable_logits` | `boolean` | No | `true` in the example | Include model logits in the response |

### Example Configuration

```json
{
  "apiKey": "<NVIDIA_API_KEY>",
  "sequence": "ATGCGTACGTTAGCTA",
  "num_tokens": 10,
  "temperature": 1,
  "top_k": 4,
  "top_p": 0.9,
  "random_seed": 42,
  "enable_sampled_probs": true,
  "enable_elapsed_ms_per_token": true,
  "pollSeconds": 2,
  "timeout": 900,
  "enable_logits": true
}
```

### Sequence Input

Provide a DNA or RNA sequence using the `sequence` parameter:

```text
ATGCGTACGTTAGCTA
```

The sequence is used as the context from which Evo2 generates additional tokens. Validate the alphabet and sequence format before sending it to the model.

### Sampling Controls

- `temperature` controls the randomness of token selection. Higher values may produce more diverse sequences.
- `top_k` restricts sampling to the K most likely candidate tokens.
- `top_p` restricts sampling to the smallest candidate set whose cumulative probability reaches the configured threshold.
- `random_seed` can be used to reproduce a generation run with the same settings.

### Response Diagnostics

Set the diagnostic flags to `true` when additional model information is needed:

- `enable_sampled_probs` includes probabilities for sampled tokens.
- `enable_elapsed_ms_per_token` includes per-token generation timing.
- `enable_logits` includes logits produced by the model.

Diagnostic data may increase response size.

### Polling and Timeout

The example polls every two seconds and allows up to 900 seconds for completion:

```json
{
  "pollSeconds": 2,
  "timeout": 900
}
```

Increase the timeout for longer generation requests and reduce polling frequency when frequent status checks are unnecessary.

### API Key and Secrets

The node requires an NVIDIA API key. The example workflow currently contains a hard-coded `apiKey` value in its node parameters.

Use a protected Fusion credential or secret reference instead of placing a real key directly in workflow JSON. Documentation and exported examples should contain only:

```text
<NVIDIA_API_KEY>
```

**Security notice:** Rotate or revoke the exposed key in `apps/automation/nodes/nvidia-bionemo-evo2-generate/example.workflow.json` before sharing or committing the workflow.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional data supplied by a preceding workflow node |

The node configuration defines the seed sequence, generation length, sampling settings, diagnostics, API key, and request controls. An upstream node can prepare a dynamic DNA or RNA sequence before execution.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Successful Evo2 generation result containing generated sequence data and optional diagnostics |
| `error` | `object` or `string` | Authentication, validation, polling, timeout, service, or network failure |

The exact success response depends on the BioNeMo Evo2 service response and the Fusion runtime. Use a Log node to inspect the returned structure during workflow development.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Generate a DNA Sequence

Generate 10 tokens from a DNA seed sequence.

```json
{
  "apiKey": "<NVIDIA_API_KEY>",
  "sequence": "ATGCGTACGTTAGCTA",
  "num_tokens": 10,
  "temperature": 1,
  "top_k": 4,
  "top_p": 0.9,
  "random_seed": 42
}
```

### Generate with Diagnostics

Include probabilities, elapsed time per token, and logits.

```json
{
  "apiKey": "<NVIDIA_API_KEY>",
  "sequence": "ATGCGTACGTTAGCTA",
  "num_tokens": 10,
  "enable_sampled_probs": true,
  "enable_elapsed_ms_per_token": true,
  "enable_logits": true
}
```

### Reproducible Generation

Set `random_seed` to repeat a generation configuration:

```json
{
  "apiKey": "<NVIDIA_API_KEY>",
  "sequence": "ATGCGTACGTTAGCTA",
  "num_tokens": 10,
  "temperature": 1,
  "top_k": 4,
  "top_p": 0.9,
  "random_seed": 42
}
```

The service and runtime must use deterministic behavior for identical results to be guaranteed.

### Dynamic Sequence Input

An upstream node can provide the seed sequence:

```json
{
  "sequence": "AUGCGUACGUUAGCUA"
}
```

Keep the API key and generation settings in protected node configuration or secret management.

### Example Timeout Case

A long generation request can exceed the configured timeout:

```json
{
  "apiKey": "<NVIDIA_API_KEY>",
  "sequence": "ATGCGTACGTTAGCTA",
  "num_tokens": 1000,
  "pollSeconds": 2,
  "timeout": 60
}
```

Inspect the `error` output and increase the timeout or reduce `num_tokens` as appropriate.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->

## Workflow Integration

### Example Workflow

The example workflow connects a Manual Trigger to the Evo2 node and sends the successful result to a Log node.

```fusion-workflow
src: example.workflow.json
title: Generate DNA or RNA sequences with Evo2
```

### Common Patterns

- **Sequence generation:** Manual Trigger → BioNeMo: Evo2 Generate → Log
- **Dynamic generation:** Data source → Function → BioNeMo: Evo2 Generate
- **Sequence validation:** Evo2 Generate → DNA/RNA validation or analysis node
- **Further processing:** Evo2 Generate → BioNeMo or bioinformatics node
- **Error handling:** Evo2 Generate `error` output → Notification or recovery node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Common Issues

#### Authentication failed

**Cause:** The NVIDIA API key is missing, invalid, expired, revoked, or unavailable to the configured service.

**Solution:** Configure a valid NVIDIA API key through protected secret management. Rotate the key if it has been exposed in a workflow file.

#### Invalid sequence input

**Cause:** The seed sequence is empty, malformed, or contains unsupported characters.

**Solution:** Validate the sequence and provide a valid DNA or RNA input before execution.

#### Generation timed out

**Cause:** The requested token count or service workload exceeded the configured `timeout`.

**Solution:** Increase `timeout`, reduce `num_tokens`, or adjust the polling settings.

#### Polling failed

**Cause:** The node could not check the generation status at the configured interval.

**Solution:** Verify network connectivity, use a reasonable `pollSeconds` value, and inspect the service response.

#### Generation failed

**Cause:** Evo2 rejected the request or could not generate a continuation from the supplied sequence.

**Solution:** Check the sequence, token count, sampling settings, and API key. Start with the example values and change one parameter at a time.

#### Response is too large

**Cause:** Logits and probability diagnostics can significantly increase response size.

**Solution:** Disable `enable_logits`, `enable_sampled_probs`, or other diagnostics when they are not needed.

#### Node is not registered

**Cause:** The package containing the `nvidia-bionemo-evo2-generate` node is not loaded or registered in the Fusion runtime.

**Solution:** Verify that the package was built, deployed, and loaded successfully.

### Error Categories

| Error | Cause | Solution |
|-------|-------|----------|
| Authentication error | Missing or invalid NVIDIA API key | Configure or rotate the protected API key |
| Invalid sequence | Malformed DNA or RNA input | Validate the sequence |
| Validation error | Missing or invalid generation parameter | Check token count and sampling controls |
| Timeout error | Generation exceeded the configured timeout | Increase `timeout` or reduce `num_tokens` |
| Polling error | Status checks could not complete | Check connectivity and `pollSeconds` |
| Generation error | Evo2 could not produce a sequence | Adjust input and sampling settings |
| Service or network error | BioNeMo service could not be reached | Check connectivity and service availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- [BioNeMo: MSA Search](../nvidia-bionemo-msa-search/en.md) - Search or analyze biological sequence data
- [BioNeMo: SMILES to SDF](../nvidia-bionemo-smiles-to-sdf/en.md) - Convert molecular SMILES into SDF structures
- [Log](./log.md) - Inspect Evo2 generation results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Added Evo2 sequence-generation documentation and security guidance |

<!-- /SECTION: changelog -->


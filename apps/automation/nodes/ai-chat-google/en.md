---
node_id: "ai-chat-google"
title: "AI Chat (Google)"
description: "Run Gemini Developer API chat models through the LangChain Google Generative AI adapter."
category: "ai"
subcategory: "agents-chat"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [google, gemini, generative-ai, langchain]
related_nodes: [google-llm, agent]
---

<!-- SECTION: overview -->
# AI Chat (Google)

> **Category:** AI&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Agent LLM Node

Send prompts to Gemini models through `@langchain/google-genai`. This node targets the Gemini Developer API and is distinct from `google-llm`, which uses the Google Auth adapter.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|---|---|---:|---|
| `model` | string | Yes | Gemini model identifier. |
| `apiKey` | string | Yes | Gemini Developer API key. |
| `temperature` | number | No | Sampling temperature. |
| `maxOutputTokens` | number | No | Maximum generated tokens. |
| `maxReasoningTokens` | number | No | Maximum reasoning-token budget for supported models. |
| `seed` | number | No | Optional deterministic sampling seed. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Prompt or chat input accepted by the workflow LLM node.
- **Success:** Model response from the LangChain adapter.
- **Error:** Authentication, validation, rate-limit, or provider error.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Ask Gemini and log the response
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store the API key as a workflow secret. Do not include confidential data in prompts unless the configured Google account and retention policy permit it.
<!-- /SECTION: security -->

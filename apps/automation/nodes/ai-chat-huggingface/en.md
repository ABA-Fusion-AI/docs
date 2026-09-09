---
node_id: "ai-chat-huggingface"
title: "AI Chat (HuggingFace)"
description: "Hugging Face models through its OpenAI-compatible Inference Providers API."
category: "ai"
subcategory: "agents-chat"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - ai
  - llm
  - huggingface
  - chat
  - completion
  - open-source
  - langchain
  - agent-tool
  - inference-providers
related_nodes:
  - agent
  - openrouter-llm
  - ai-chat-openrouter
  - ai-chat-google
  - function
---

<!-- SECTION: overview -->
# AI Chat (HuggingFace)

> **Category:** AI &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action / Agent Tool Node

The **AI Chat (HuggingFace)** node connects workflows and autonomous agents to thousands of open-source and proprietary models hosted on [Hugging Face](https://huggingface.co) through the **Inference Providers** API — an OpenAI-compatible endpoint powered by multiple compute back-ends (AWS, Google Cloud, Azure, and more). Built on LangChain's `ChatOpenAI` adapter (`@langchain/openai`), it requires no extra SDKs and works with any model available through Hugging Face's router.

This node can be used in two primary ways:
1. **As an Action Node:** Receives a prompt or input messages directly in sequential automation pipelines and outputs the model's chat completion.
2. **As an Agent Tool:** Connects to an **Agent** node via the `tool` handle, allowing autonomous agents to query Hugging Face models on demand during multi-step reasoning tasks.

### Key Use Cases

- **Open-Source Model Access:** Run leading open-weights models — Llama 3.3, Mistral, Gemma, Qwen, DeepSeek — through managed infrastructure without self-hosting.
- **Agent Tool Calling:** Equip AI Agents with Hugging Face model access to generate reports, draft content, translate text, or perform domain-specific analysis.
- **Cost-Effective Inference:** Access high-quality open-source models at competitive pricing through Hugging Face's multi-provider routing.
- **Model Experimentation:** Rapidly prototype and compare thousands of models from the Hugging Face Hub by simply changing the `model` parameter.
- **Privacy-Conscious Workflows:** Choose specific inference providers and regions to meet data residency or compliance requirements.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `model` | `string` | No | `meta-llama/Llama-3.3-70B-Instruct` | The Hugging Face model identifier (repo ID). Browse models at [huggingface.co/models](https://huggingface.co/models?pipeline_tag=text-generation). |
| `apiKey` | `string` | No | — | Your Hugging Face API token (`hf_...`). Create one at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens). Store securely in workflow secrets. |
| `baseUrl` | `string` | No | `https://router.huggingface.co/v1` | Base URL for the Hugging Face Inference Providers API. Change only when routing via a specific provider endpoint or custom proxy. |
| `temperature` | `number` | No | — | Sampling temperature controlling creativity and randomness (`0.0` = deterministic, `2.0` = highly creative). |
| `streaming` | `boolean` | No | `false` | Enable or disable token streaming. When `false`, the node returns the full completed response at once. |
| `systemMessage` | `string` | No | — | System prompt setting the assistant's persona, role, instructions, and output constraints. |
| `maxTokens` | `number` | No | — | Maximum number of tokens the model is allowed to generate in its response. |
| `topP` | `number` | No | — | Nucleus sampling probability cutoff (e.g., `0.9`). An alternative to temperature for controlling diversity. |
| `maxConcurrency` | `number` | No | — | Maximum number of concurrent requests the underlying client can dispatch simultaneously. |
| `timeout` | `number` | No | — | Maximum request duration in milliseconds before timing out and emitting an error. |

### Popular Model Identifiers

Hugging Face uses the standard `owner/model-name` repo ID format:

| Model Identifier | Developer | Strengths / Best For |
|------------------|-----------|----------------------|
| `meta-llama/Llama-3.3-70B-Instruct` | Meta | **Default.** Powerful instruction-tuned model for general chat, reasoning, and coding. |
| `meta-llama/Llama-3.1-8B-Instruct` | Meta | Lightweight, fast model ideal for latency-sensitive tasks and cost-efficient pipelines. |
| `mistralai/Mistral-Small-24B-Instruct-2501` | Mistral AI | Compact, strong multilingual model with structured output support. |
| `Qwen/Qwen2.5-72B-Instruct` | Alibaba | Excellent multilingual capabilities with strong reasoning and math performance. |
| `deepseek-ai/DeepSeek-R1` | DeepSeek | Advanced chain-of-thought reasoning model for math, code, and logic. |
| `google/gemma-2-27b-it` | Google | Open-weights model with strong instruction following and safety alignment. |
| `NousResearch/Hermes-3-Llama-3.1-8B` | Nous Research | Fine-tuned for function calling, structured output, and agentic workflows. |

> Discover thousands of available models at [huggingface.co/models](https://huggingface.co/models?pipeline_tag=text-generation&sort=trending).

### `temperature` Guidelines

| Temperature | Behavior | Recommended Use Cases |
|-------------|----------|-----------------------|
| `0.0 – 0.2` | Focused, deterministic, consistent | Data extraction, classification, code syntax, structured JSON output |
| `0.5 – 0.7` | Balanced creativity and factual accuracy | General Q&A, conversational agents, summarization, email drafting |
| `0.8 – 1.2` | Creative and diverse responses | Brainstorming, copywriting, creative writing, marketing slogans |
| `> 1.2` | Highly divergent | Experimental generation, radical idea generation |

### Inference Providers

Hugging Face routes requests through multiple inference providers behind `router.huggingface.co`. The router automatically selects the optimal provider, but you can also target a specific one by changing `baseUrl`:

| Provider | Endpoint Example | Notes |
|----------|------------------|-------|
| **Auto (Default)** | `https://router.huggingface.co/v1` | Automatic provider selection based on model availability and latency. |
| **Dedicated** | Varies per deployment | For Hugging Face Inference Endpoints (dedicated GPU instances). |

> Learn more about inference providers at [huggingface.co/docs/api-inference](https://huggingface.co/docs/api-inference).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | The user prompt string or structured messages array to send to the model. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Chat completion response containing the generated text, metadata, and token usage. |
| `error` | `object` | Emitted when authentication fails, timeout occurs, or the provider returns an error. |

### Output Schema (`success`)

```json
{
  "id": "chatcmpl-abc123xyz",
  "model": "meta-llama/Llama-3.3-70B-Instruct",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Here is the response generated by the model..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 38,
    "completion_tokens": 112,
    "total_tokens": 150
  }
}
```

### Accessing Output in Expressions

Downstream nodes can reference the output using Fusion expressions:

- **Generated Content:**
  ```
  {{ outputs["AI Chat (HuggingFace)"].success.choices[0].message.content }}
  ```
- **Total Tokens Used:**
  ```
  {{ outputs["AI Chat (HuggingFace)"].success.usage.total_tokens }}
  ```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: AI Agent with HuggingFace Chat Tool
```

### How It Works

The provided example demonstrates using **AI Chat (HuggingFace)** as an intelligent tool attached to an **Agent**:

1. **Manual Trigger (`manual-trigger`):** Starts the workflow execution manually.
2. **OpenRouter LLM (`openrouter-llm`):** Connected via the `llm` port to provide the core reasoning brain for the Agent.
3. **AI Chat (HuggingFace) (`ai-chat-huggingface`):** Connected via the `tool` port to the Agent. It acts as an on-demand chat tool that the Agent can invoke to run completions, generate reports, or produce specialized outputs using Hugging Face models.
4. **Agent (`agent`):** Receives the user prompt, plans the execution, invokes the HuggingFace chat tool, and synthesizes the final answer.
5. **Log (`log`):** Receives the Agent's `success` output and displays the generated result for inspection.

### Alternative: Standalone Action Workflow

For direct prompt-to-response automation without an autonomous agent:

1. Connect a trigger (e.g., **Manual Trigger**, **Webhook**, or **Cron**) to `AI Chat (HuggingFace)`.
2. Map your prompt to the `input` port.
3. Configure `model`, `apiKey`, and optional `systemMessage` in the node's settings panel.
4. Connect the `success` output to downstream nodes (e.g., **Slack**, **Email Send**, or **Log**).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security & Best Practices

- **Never Hardcode API Keys:** Always store your Hugging Face token in Fusion **Secrets** (e.g., `{{ secrets.HUGGINGFACE_API_KEY }}`) or reference it via environment variables.
- **Export Hygiene:** Before exporting workflows for documentation or sharing, always verify that `parameters.apiKey`, `secrets`, and `variables` are stripped.
- **Token Permissions:** Use [fine-grained tokens](https://huggingface.co/settings/tokens) scoped to `read` access only for inference. Avoid using write-scoped tokens in production workflows.
- **Rate Limits:** Free-tier Hugging Face accounts have rate limits on inference. For production workloads, consider a [PRO subscription](https://huggingface.co/pricing) or dedicated Inference Endpoints.
- **Data Privacy:** Review the [Hugging Face Privacy Policy](https://huggingface.co/privacy) and the specific inference provider's data handling policies when processing sensitive data.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues and Solutions

#### `401 Unauthorized`
- **Cause:** The `apiKey` is invalid, empty, expired, or lacks the required permissions.
- **Solution:** Verify your token at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens). Ensure it starts with `hf_` and has at least `read` permission.

#### `429 Too Many Requests`
- **Cause:** Rate limit exceeded on the free tier or the selected inference provider.
- **Solution:** Wait briefly and retry, or upgrade to a [Hugging Face PRO plan](https://huggingface.co/pricing) for higher rate limits.

#### `404 Model Not Found` or `Model Not Supported`
- **Cause:** The model identifier is incorrect, or the model is not available through the Inference Providers API.
- **Solution:** Confirm the exact repo ID at [huggingface.co/models](https://huggingface.co/models?pipeline_tag=text-generation). Not all models on the Hub are served by the inference API — check for the "Inference Providers" badge on the model card.

#### Empty or Truncated Response
- **Cause:** `maxTokens` is set too low for the expected output length.
- **Solution:** Increase `maxTokens` or remove the setting to let the model use its default limit.

#### Request Timeout
- **Cause:** The model is cold-starting (loading into GPU memory) or the prompt is too complex for the timeout window.
- **Solution:** Increase the `timeout` value (e.g., `120000` for 120 seconds). First requests to less-popular models may require a cold start. Retry after a few seconds.

#### `streaming` Output Issues
- **Cause:** `streaming: true` is enabled, but downstream nodes expect a single completed JSON response.
- **Solution:** Keep `streaming: false` unless downstream nodes are specifically configured to consume real-time streaming chunks.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Agent](../agent/en.md) – Autonomous agent capable of reasoning and orchestrating tools.
- [AI Chat (OpenRouter)](../ai-chat-openrouter/en.md) – Access 200+ models via OpenRouter's unified gateway.
- [AI Chat (Google)](../ai-chat-google/en.md) – Direct Gemini chat completions via the Gemini Developer API.
- [Mistral LLM](../mistral-llm/en.md) – Dedicated Mistral AI LLM node.
- [Function](../function/en.md) – Transform prompts or parse structured completion outputs.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Initial documentation with full parameter reference, agent tool integration, and example workflow. |

<!-- /SECTION: changelog -->

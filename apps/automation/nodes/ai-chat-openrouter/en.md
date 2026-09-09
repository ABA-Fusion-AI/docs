---
node_id: "ai-chat-openrouter"
title: "AI Chat (OpenRouter)"
description: "OpenRouter models through its OpenAI-compatible API."
category: "ai"
subcategory: "agents-chat"
version: "1.0.0"
language: "en"
last_updated: "2026-09-09"
author: "Fusion Team"
tags:
  - ai
  - llm
  - openrouter
  - chat
  - completion
  - multi-model
  - langchain
  - agent-tool
related_nodes:
  - agent
  - openrouter-llm
  - ai-chat-google
  - ai-chat-mistral
  - function
---

<!-- SECTION: overview -->
# AI Chat (OpenRouter)

> **Category:** AI &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action / Agent Tool Node

The **AI Chat (OpenRouter)** node connects workflows and autonomous agents to **200+ AI models** through [OpenRouter](https://openrouter.ai)'s unified OpenAI-compatible API gateway. Built on LangChain's `ChatOpenAI` adapter (`@langchain/openai`), it enables flexible interaction with leading models from Meta, Anthropic, OpenAI, Google, Mistral, and DeepSeek without requiring individual provider SDKs.

This node can be used in two primary ways:
1. **As an Action Node:** Receives a prompt or input messages directly in sequential automation pipelines and outputs the model's chat completion.
2. **As an Agent Tool:** Connects to an **Agent** node via the `tool` handle, allowing autonomous agents to query OpenRouter models on demand during multi-step reasoning tasks.

### Key Use Cases

- **Agent Tool Calling:** Equip AI Agents with OpenRouter model access to run specialized tasks, draft reports, perform translations, or query domain-specific models.
- **Multi-Model Pipelines:** Easily compare or switch between models (e.g., Llama 3.3, Claude 3.5 Sonnet, GPT-4o, DeepSeek) simply by changing the `model` parameter.
- **Cost-Optimized Inference:** Route prompts to high-performance, cost-effective models like DeepSeek V3/R1 or open-weights Llama models without managing infrastructure.
- **Unified Gateway & Fallbacks:** Benefit from OpenRouter's built-in provider load-balancing and automated fallback routing.
- **Structured Content Generation:** Generate structured JSON, summarize documents, translate text, or answer user inquiries with customized system instructions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|---------|-------------|
| `model` | `string` | No | `meta-llama/llama-3.3-70b-instruct` | The model identifier in `provider/model-name` format. Browse available models at [openrouter.ai/models](https://openrouter.ai/models). |
| `apiKey` | `string` | No | — | Your OpenRouter API key (`sk-or-v1-...`). Obtain one at [openrouter.ai/keys](https://openrouter.ai/keys). Store securely in workflow secrets. |
| `baseUrl` | `string` | No | `https://openrouter.ai/api/v1` | Base URL for the OpenRouter API. Change only when routing via a custom proxy or gateway. |
| `temperature` | `number` | No | — | Sampling temperature controlling creativity and randomness (`0.0` = deterministic, `2.0` = highly creative). |
| `streaming` | `boolean` | No | `false` | Enable or disable token streaming. When `false`, the node returns the full completed response at once. |
| `systemMessage` | `string` | No | — | System prompt setting the assistant's persona, role, instructions, and output constraints. |
| `maxTokens` | `number` | No | — | Maximum number of tokens the model is allowed to generate in its response. |
| `topP` | `number` | No | — | Nucleus sampling probability cutoff (e.g., `0.9`). An alternative to temperature for controlling diversity. |
| `maxConcurrency` | `number` | No | — | Maximum number of concurrent requests the underlying client can dispatch simultaneously. |
| `timeout` | `number` | No | — | Maximum request duration in milliseconds before timing out and emitting an error. |

### Popular Model Identifiers

OpenRouter identifies models using the format `provider/model-name`:

| Model Identifier | Provider | Strengths / Best For |
|------------------|----------|----------------------|
| `meta-llama/llama-3.3-70b-instruct` | Meta | **Default.** Versatile, powerful open-weights model for general chat and reasoning. |
| `deepseek/deepseek-chat` | DeepSeek | Exceptionally fast, cost-effective for general instructions and code. |
| `deepseek/deepseek-r1` | DeepSeek | Advanced chain-of-thought mathematical, logical, and code reasoning. |
| `anthropic/claude-3.5-sonnet` | Anthropic | Industry-leading coding, nuances, and detailed analytical reasoning. |
| `openai/gpt-4o` | OpenAI | Flagship multimodal intelligence with high speed and broad capabilities. |
| `google/gemini-2.0-flash-001` | Google | Ultra-fast multimodal model with massive context capabilities. |
| `mistralai/mistral-large-2411` | Mistral | Strong multilingual capabilities and structured output adherence. |

> Discover the full catalog of models, pricing, and context limits at [openrouter.ai/models](https://openrouter.ai/models).

### `temperature` Guidelines

| Temperature | Behavior | Recommended Use Cases |
|-------------|----------|-----------------------|
| `0.0 – 0.2` | Focused, deterministic, consistent | Data extraction, classification, code syntax, structured JSON output |
| `0.5 – 0.7` | Balanced creativity and factual accuracy | General Q&A, conversational agents, summarization, email drafting |
| `0.8 – 1.2` | Creative and diverse responses | Brainstorming, copywriting, creative writing, marketing slogans |
| `> 1.2` | Highly divergent | Experimental generation, radical idea generation |

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
  "id": "gen-1741518000-abcdef123456",
  "model": "meta-llama/llama-3.3-70b-instruct",
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
    "prompt_tokens": 42,
    "completion_tokens": 128,
    "total_tokens": 170
  }
}
```

### Accessing Output in Expressions

Downstream nodes can reference the output using Fusion expressions:

- **Generated Content:**
  ```
  {{ outputs["AI Chat (OpenRouter)"].success.choices[0].message.content }}
  ```
- **Total Tokens Used:**
  ```
  {{ outputs["AI Chat (OpenRouter)"].success.usage.total_tokens }}
  ```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: AI Agent with OpenRouter Chat Tool
```

### How It Works

The provided example demonstrates using **AI Chat (OpenRouter)** as an intelligent tool attached to an **Agent**:

1. **Manual Trigger (`manual-trigger`):** Starts the workflow execution manually.
2. **OpenRouter LLM (`openrouter-llm`):** Connected via the `llm` port to provide the core reasoning brain for the Agent.
3. **AI Chat (OpenRouter) (`ai-chat-openrouter`):** Connected via the `tool` port to the Agent. It acts as an on-demand chat tool that the Agent can invoke to run completions or generate dedicated reports.
4. **Agent (`agent`):** Receives the user prompt, plans the execution, invokes the OpenRouter chat tool, and synthesizes the final answer.
5. **Log (`log`):** Receives the Agent's `success` output and displays the generated result for inspection.

### Alternative: Standalone Action Workflow

For direct prompt-to-response automation without an autonomous agent:

1. Connect a trigger (e.g., **Manual Trigger**, **Webhook**, or **Cron**) to `AI Chat (OpenRouter)`.
2. Map your prompt to the `input` port.
3. Configure `model`, `apiKey`, and optional `systemMessage` in the node's settings panel.
4. Connect the `success` output to downstream nodes (e.g., **Slack**, **Email Send**, or **Log**).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security & Best Practices

- **Never Hardcode API Keys:** Always store your OpenRouter API key in Fusion **Secrets** (e.g., `{{ secrets.OPENROUTER_API_KEY }}`) or reference it via environment variables.
- **Export Hygiene:** Before exporting workflows for documentation or sharing, always verify that `parameters.apiKey`, `secrets`, and `variables` are stripped.
- **Key Scoping & Limits:** Use the [OpenRouter Key Management](https://openrouter.ai/settings/keys) console to set credit limits and restrict allowed models to prevent unexpected expenditures.
- **Data Privacy:** Be mindful of data privacy policies when routing sensitive customer data through external third-party model providers.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues and Solutions

#### `401 Unauthorized`
- **Cause:** The `apiKey` is invalid, empty, expired, or revoked.
- **Solution:** Verify your API key at [openrouter.ai/keys](https://openrouter.ai/keys). Ensure it starts with `sk-or-v1-` and is properly configured in your workflow secrets.

#### `402 Payment Required`
- **Cause:** Insufficient OpenRouter account balance or credit limit reached.
- **Solution:** Check your credits and usage limits on the [OpenRouter Account Dashboard](https://openrouter.ai/credits).

#### `404 Model Not Found`
- **Cause:** The model identifier is mistyped or no longer available on OpenRouter.
- **Solution:** Confirm the exact format (`provider/model-name`) against the [OpenRouter Models List](https://openrouter.ai/models).

#### Request Timeout
- **Cause:** The requested model (especially large 70B+ or reasoning models like DeepSeek R1) takes longer than the default timeout to generate responses.
- **Solution:** Increase the `timeout` parameter (in milliseconds, e.g., `60000` for 60 seconds), or choose a lower-latency model such as `deepseek/deepseek-chat` or `google/gemini-2.0-flash-001`.

#### `streaming` Output Issues
- **Cause:** `streaming: true` is enabled, but downstream nodes expect a single completed JSON response.
- **Solution:** Keep `streaming: false` unless downstream nodes are specifically configured to consume real-time streaming chunks.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Agent](../agent/en.md) – Autonomous agent capable of reasoning and orchestrating tools.
- [OpenRouter LLM](../openrouter-llm/en.md) – LangChain LLM provider node specifically tailored as an Agent brain.
- [AI Chat (Google)](../ai-chat-google/en.md) – Direct Gemini chat completions via the Gemini Developer API.
- [Mistral LLM](../mistral-llm/en.md) – Dedicated Mistral AI LLM node.
- [Function](../function/en.md) – Transform prompts or parse structured completion outputs.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-09 | Updated documentation with agent tool integration, complete parameter reference, and example workflow alignment. |

<!-- /SECTION: changelog -->

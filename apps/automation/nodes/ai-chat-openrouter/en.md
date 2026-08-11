---
node_id: "ai-chat-openrouter"
title: "AI Chat (OpenRouter)"
description: "Access 200+ AI models through OpenRouter's OpenAI-compatible API."
category: "ai"
subcategory: "Chat & Completion"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - ai
  - llm
  - openrouter
  - chat
  - completion
  - multi-model
related_nodes:
  - ai-chat
  - mistral-llm
  - agent
  - function
---

<!-- SECTION: overview -->
# AI Chat (OpenRouter)

> **Category:** AI &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send chat completion requests to **200+ AI models** through [OpenRouter](https://openrouter.ai) — a unified API gateway that routes to providers like Meta, Anthropic, Google, Mistral, OpenAI, and more. Uses an OpenAI-compatible API format, so no per-provider SDK setup is needed.

### Use Cases

- **Multi-Model Pipelines:** Route requests to different models (fast vs. powerful) based on task complexity, without changing the node configuration.
- **Cost-Optimized AI:** Use OpenRouter's routing to automatically select the cheapest model that meets your quality threshold.
- **Open-Source Models:** Access models like Llama 3.3, Mistral, Gemma, or DeepSeek without managing your own infrastructure.
- **Content Generation:** Generate text, summaries, translations, or structured data from any prompt using any supported model.
- **Fallback & Redundancy:** Configure OpenRouter's automatic fallback to switch models if the primary provider is unavailable.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `model` | `string` | No | `meta-llama/llama-3.3-70b-instruct` | The model identifier in `provider/model-name` format. See [openrouter.ai/models](https://openrouter.ai/models) for the full list. |
| `apiKey` | `string` | No | — | Your OpenRouter API key. Obtain one at [openrouter.ai/keys](https://openrouter.ai/keys). Store as a workflow secret. |
| `baseUrl` | `string` | No | `https://openrouter.ai/api/v1` | The OpenRouter API base URL. Change only if using a custom proxy or endpoint. |
| `temperature` | `number` | No | — | Controls output randomness. Range: `0.0` (deterministic) to `2.0` (highly creative). Default varies by model. |
| `streaming` | `boolean` | No | `false` | Enable streaming responses. When enabled, the node emits partial chunks as the model generates them. |
| `systemMessage` | `string` | No | — | System-level instruction that sets the model's role, persona, and behavior for the conversation. |
| `maxTokens` | `number` | No | — | Maximum number of tokens in the model's response. Limits output length and cost. |
| `topP` | `number` | No | — | Nucleus sampling threshold. Lower values produce more focused outputs. |
| `maxConcurrency` | `number` | No | — | Maximum number of simultaneous requests this node can process in a parallel execution context. |
| `timeout` | `number` | No | — | Request timeout in milliseconds. If the model does not respond within this time, an error is emitted. |

### Model Format

OpenRouter uses a `provider/model-name` format for model identifiers:

| Example Model | Provider | Notes |
|---------------|----------|-------|
| `meta-llama/llama-3.3-70b-instruct` | Meta | Default. Large, capable open-source model. |
| `anthropic/claude-3.5-sonnet` | Anthropic | Strong reasoning and instruction following. |
| `google/gemini-pro-1.5` | Google | Long context window, multimodal. |
| `mistralai/mistral-large` | Mistral | Fast and multilingual. |
| `openai/gpt-4o` | OpenAI | Flagship multimodal model. |
| `deepseek/deepseek-chat` | DeepSeek | Cost-efficient, strong on code and reasoning. |

> Browse the full list of 200+ supported models at [openrouter.ai/models](https://openrouter.ai/models).

### `temperature` Guide

| Value | Behavior | Best for |
|-------|----------|----------|
| `0.0` | Fully deterministic | Factual extraction, structured output, JSON generation |
| `0.5–0.7` | Balanced | General Q&A, summarization, translation |
| `1.0–1.5` | Creative | Copywriting, brainstorming, storytelling |
| `2.0` | Highly random | Experimental or creative generation |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | The user message to send to the model, or a full messages array for multi-turn conversations. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The model's response in OpenAI-compatible chat completion format. |
| `error` | `Error` | Emitted on authentication errors, model errors, timeout, or invalid input. |

### Output Schema (`success`)

```json
{
  "id": "gen-abc123",
  "model": "meta-llama/llama-3.3-70b-instruct",
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Here is a summary of the topic you requested..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 128,
    "completion_tokens": 256,
    "total_tokens": 384
  }
}
```

### Accessing the Response Text

Use an expression to extract the model's reply from the output:

```
{{ outputs["AI Chat (OpenRouter)"].success.choices[0].message.content }}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Chat Completion with OpenRouter
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Function Node:** Prepares the user message or prompt.
3. **AI Chat (OpenRouter) Node:** Sends the message to the configured model via OpenRouter and returns the completion.
4. **Log Node:** Displays the model's response.

### Common Patterns

- **Simple Prompt:** Pass a plain string as input and the node wraps it in a user message automatically.
- **System + User:** Set `systemMessage` to define the assistant's role, and pass the user's question as input.
- **Model Comparison:** Clone the workflow, change only the `model` parameter, and compare outputs side-by-side.
- **Structured Output:** Prompt the model to return JSON, then pass the response to a Parse JSON node for downstream processing.
- **Cost Routing:** Use `openrouter/auto` as the model to let OpenRouter select the optimal model based on cost and capability for each request.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store your `apiKey` in Fusion's **Secrets** system. Do not hardcode it in workflow parameters or export it in workflow files.

- OpenRouter API keys can be scoped with **credit limits** and **model restrictions** from the [OpenRouter dashboard](https://openrouter.ai/settings/keys).
- Use separate API keys for development and production workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Unauthorized` — Invalid API key
- **Cause:** The `apiKey` is missing, incorrect, or has been revoked.
- **Solution:** Verify your key at [openrouter.ai/keys](https://openrouter.ai/keys) and ensure it is correctly set in the node or workflow secrets.

#### `Model not found` or `Provider error`
- **Cause:** The model identifier is incorrect or the model is temporarily unavailable on OpenRouter.
- **Solution:** Check the model ID format (`provider/model-name`) at [openrouter.ai/models](https://openrouter.ai/models). OpenRouter can automatically fall back to alternative providers if the primary is down.

#### Empty or truncated response
- **Cause:** `maxTokens` is set too low for the expected output length.
- **Solution:** Increase `maxTokens` or remove it to let the model use its default limit.

#### Request timeout
- **Cause:** The model is taking too long to respond, or `timeout` is set too low.
- **Solution:** Increase the `timeout` value. Large models (70B+) can take several seconds for complex prompts. Consider switching to a faster model for latency-sensitive workflows.

#### `streaming` output not handled
- **Cause:** `streaming: true` is enabled but the downstream node expects a complete response object.
- **Solution:** Disable `streaming` unless your downstream logic is designed to handle partial chunk events.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [AI Chat](./ai-chat.md) – Direct AI chat with a specific provider (OpenAI, Anthropic, etc.)
- [Mistral LLM](./mistral-llm.md) – Use Mistral AI models as an Agent backbone
- [Agent](./agent.md) – Autonomous AI agent powered by any LLM node
- [Function](./function.md) – Build dynamic prompts or parse structured model output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial documentation |

<!-- /SECTION: changelog -->

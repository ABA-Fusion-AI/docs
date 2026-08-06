---
node_id: "mistral-llm"
title: "Mistral LLM"
description: "A node that uses Mistral AI's LLM models to power agents with state-of-the-art language understanding and generation."
category: "ai"
subcategory: "Mistral AI"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - mistral
  - llm
  - ai
  - agent
  - language-model
  - generative-ai
related_nodes:
  - agent
  - ai-chat
  - function
  - http-request
---

<!-- SECTION: overview -->
# Mistral LLM

> **Category:** AI &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Agent LLM Node

Connect Mistral AI's large language models to your Fusion agents. The **Mistral LLM** node acts as the intelligence backbone of an AI Agent node — it provides the underlying language model that the agent uses to reason, plan, and generate responses. Configure the model, system prompt, temperature, and other generation parameters directly from the node panel.

### Use Cases

- **Intelligent Automation Agents:** Attach Mistral models to Agent nodes to enable complex reasoning over user inputs, structured data, or external API results.
- **Domain-Specific Assistants:** Configure a custom `SystemPrompt` to create specialized agents (legal, medical, educational, customer support, etc.).
- **Multilingual Content Generation:** Leverage Mistral's multilingual capabilities to generate, translate, or summarize content in multiple languages.
- **RAG Pipelines:** Combine with memory and retrieval nodes to build Retrieval-Augmented Generation (RAG) workflows.
- **Code Generation & Review:** Use Mistral Codestral or reasoning models to generate, review, or explain code snippets within automated pipelines.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `model` | `string` | No | `mistral-large-latest` | The Mistral model identifier to use. See available models below. |
| `apiKey` | `string` | Yes | — | Your Mistral AI API key. Obtain one at [console.mistral.ai](https://console.mistral.ai). |
| `temperature` | `number` | No | `0.7` | Controls randomness of the output. Range: `0.0` (deterministic) to `1.0` (creative). |
| `systemPrompt` | `string` | No | — | A system-level instruction that shapes the agent's behavior, role, and tone throughout the conversation. |
| `maxTokens` | `number` | No | — | Maximum number of tokens the model can generate in a single response. |
| `topP` | `number` | No | `1.0` | Nucleus sampling probability. Values below `1.0` restrict the token pool to the top cumulative probability mass. |
| `maxRetries` | `number` | No | `3` | Number of retry attempts on transient API failures before the node emits an error. |

### Available Models

| Model ID | Description |
|----------|-------------|
| `mistral-large-latest` | Most capable flagship model for complex reasoning and instruction-following tasks. |
| `mistral-medium-latest` | Balanced model for general-purpose tasks with strong performance at lower cost. |
| `mistral-small-latest` | Lightweight and fast model ideal for simple tasks and high-throughput workloads. |
| `codestral-latest` | Specialized model for code generation, completion, and explanation. |
| `mistral-embed` | Embedding model for semantic search and RAG pipelines. |
| `open-mistral-7b` | Open-weight 7B model, good for self-hosted or cost-sensitive scenarios. |
| `open-mixtral-8x7b` | Open-weight Mixture of Experts model with high throughput and broad capability. |

> **Tip:** Always use the `*-latest` aliases in production to automatically benefit from model improvements without changing your workflow configuration.

### Parameter Details

#### `temperature`

Controls how deterministic or creative the model's responses are:

- `0.0` — Fully deterministic, always picks the most likely token. Best for factual Q&A and structured extraction.
- `0.5` — Balanced output. Suitable for most general-purpose tasks.
- `1.0` — Highly creative and varied. Best for brainstorming, storytelling, or content generation.

#### `systemPrompt`

The system prompt defines the agent's persona, task scope, and constraints. It is injected at the start of every conversation turn. Example:

```
You are a senior legal analyst. Your task is to review contract clauses,
identify risks, and summarize findings in plain English.
Always cite the relevant clause number in your response.
```

#### `topP`

Works in conjunction with `temperature`. Setting `topP` to `0.9` means the model only samples from the top 90% of the probability distribution. Lower values produce more focused, coherent outputs.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

> The Mistral LLM node connects to an **Agent** node via a special `llm` connection (not a standard data connection). It does not process data directly — instead, it provides the language model backend that the Agent node calls during execution.

### Connection Types

| Connection | Type | Description |
|------------|------|-------------|
| `llm` (out) | `agentConnection` | Links the Mistral LLM provider to an Agent node's `llm` input slot. |
| `input` | `object` | Standard data input used in direct invocation scenarios. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the model call completes successfully. Contains the model's response. |
| `error` | `Error` | Emitted when the API call fails after all retries are exhausted. |

### Output Schema (`success`)

```json
{
  "content": "La Marche Verte de 1975 est une etape fondatrice de l'histoire du Maroc moderne...",
  "model": "mistral-large-latest",
  "usage": {
    "prompt_tokens": 512,
    "completion_tokens": 1024,
    "total_tokens": 1536
  },
  "finish_reason": "stop"
}
```

### Output Schema (`error`)

```json
{
  "success": false,
  "error": {
    "message": "Unauthorized: Invalid API key",
    "code": 401
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: AI History Teacher Powered by Mistral LLM
```

### How it flows

1. **Manual Trigger:** Kicks off the workflow execution on demand.
2. **Function Node:** Returns the topic to be analyzed (e.g., `"la marche vert au maroc"`).
3. **Mistral LLM Node:** Connected to the Agent node via an `agentConnection` on the `llm` port — provides `mistral-large-latest` as the reasoning engine.
4. **Agent Node:** Receives the topic as input and uses the Mistral LLM to generate a detailed, structured historical explanation according to a configured system prompt.
5. **Log Node:** Displays the agent's full generated response.

### Common Patterns

- **LLM + Agent:** The primary pattern — connect Mistral LLM to an Agent node via the `llm` port to give the agent its reasoning capabilities.
- **Multi-LLM Switching:** Swap between different LLM nodes (Mistral, OpenAI, Gemini) on the same agent to A/B test model quality and cost.
- **Dynamic Prompt Building:** Use a Function node upstream to dynamically construct a `systemPrompt` string from user data, then pass it as an expression to the Mistral LLM node.
- **Chained Agents:** Connect the output of one agent to another, each backed by different Mistral models (e.g., `mistral-small-latest` for fast extraction and `mistral-large-latest` for deep reasoning).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Unauthorized (401)`
- **Cause:** The `apiKey` parameter is missing, incorrect, or expired.
- **Solution:** Verify your API key at [console.mistral.ai](https://console.mistral.ai) and ensure it is correctly set. Use workflow secrets to store sensitive keys instead of hardcoding them.

#### `Model Not Found (404)`
- **Cause:** The model identifier in the `model` parameter does not match a valid Mistral model name.
- **Solution:** Use one of the supported model IDs listed in the configuration section. Prefer `*-latest` aliases over specific version strings.

#### `Rate Limit Exceeded (429)`
- **Cause:** Too many requests are being sent to the Mistral API within a short time window.
- **Solution:** Reduce workflow execution frequency, increase `maxRetries` with a delay node, or upgrade your Mistral plan for higher rate limits.

#### `Context Length Exceeded`
- **Cause:** The combined length of the system prompt, conversation history, and user input exceeds the model's maximum context window.
- **Solution:** Reduce the `systemPrompt` length, truncate input data in a Function node before passing it to the agent, or switch to a model with a larger context window.

#### `Agent Not Responding / Empty Output`
- **Cause:** The `systemPrompt` may be overly restrictive, or `temperature` is set to `0` with a prompt that has no clear deterministic answer.
- **Solution:** Review the system prompt for conflicting instructions. Increase `temperature` slightly and ensure the agent input contains actionable content.

### Error Codes Summary

| Error Message | HTTP Code | Solution |
|---------------|-----------|----------|
| `Unauthorized` | 401 | Verify and update the API key |
| `Model Not Found` | 404 | Use a valid model ID from the supported list |
| `Rate Limit Exceeded` | 429 | Add delay between requests or upgrade plan |
| `Internal Server Error` | 500 / 503 | Check [status.mistral.ai](https://status.mistral.ai) and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Agent](./agent.md) – The AI Agent node that uses Mistral LLM as its intelligence backbone
- [AI Chat](./ai-chat.md) – Direct conversational interface with Mistral and other LLM providers
- [Function](./function.md) – Dynamically build prompts or preprocess data before sending to the agent
- [HTTP Request](./http-request.md) – Call the Mistral REST API directly for advanced use cases
- [Google LLM](./google-llm.md) – Alternative LLM provider using Google Gemini models

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial documentation for Mistral LLM node |

<!-- /SECTION: changelog -->

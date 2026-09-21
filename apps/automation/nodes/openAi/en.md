---
node_id: "openai-llm"

title: "OpenAI LLM"

description: "Use OpenAI LLM models through LangChain ChatOpenAI."

category: "AI / LLM"

version: "1.0.0"

language: "en"

last_updated: "2026-09-21"

author: "Fusion Team"

tags:

- openai

- llm

- ai

- langchain

- chatopenai

related_nodes:

- ai-chat-openai

- ai-chat-openai-like

- function

---

**# OpenAI LLM**

> **\*\*Category:\*\*** ai-nodes | **\*\*Type:\*\*** LLM Node

Connect workflows to OpenAI LLM models using LangChain's `ChatOpenAI` client.

The **\*\*OpenAI LLM\*\*** node extends `LlmNode` and provides configuration for the model, API key, temperature, system prompt, maximum tokens, request timeout, and retry count.

**### Supported Features**

\- Configure an OpenAI model

\- Configure an OpenAI API key

\- Configure sampling temperature

\- Configure a system prompt

\- Configure maximum generated tokens

\- Configure request timeout

\- Configure maximum retries

\- Use LangChain `ChatOpenAI`

**### Use Cases**

\- Add OpenAI LLM capabilities to workflows

\- Generate text with OpenAI models

\- Configure reusable system instructions

\- Control generation length and temperature

\- Configure retry behavior

\- Use an OpenAI model as an LLM dependency

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `model` | `string` | ❌ No | `"gpt-4o-mini"` | Model name. Examples in the schema include `gpt-4o`, `gpt-4o-mini`, `gpt-4.1`, `gpt-4.1-mini`, `gpt-5-mini`, and `o3-mini`. |
| `apiKey` | `string` | ❌ No | — | OpenAI API key. |
| `temperature` | `number` | ❌ No | — | Sampling temperature. The schema description specifies `0.0` to `2.0`. |
| `systemPrompt` | `string` | ❌ No | — | System instructions or prompt. |
| `maxTokens` | `number` | ❌ No | — | Maximum tokens to generate. |
| `timeout` | `number` | ❌ No | — | Request timeout in milliseconds. |
| `maxRetries` | `number` | ❌ No | `3` | Maximum number of retries for failed requests. |

Default model:

```text
gpt-4o-mini
```

Default maximum retries:

```text
3
```

**---**

**## Operations**

This class does not define an `operation` field or an operation switch.

It extends:

```text
LlmNode<Parameters>
```

The LLM execution lifecycle is inherited from `LlmNode`.

**---**

**## Model Construction**

The LLM is created using:

```ts
protected llmFactory = (parameters: Record<string, unknown>) => {
  return new ChatOpenAI(parameters) as any;
};
```

The supplied parameter object is passed directly to:

```text
ChatOpenAI
```

No properties are removed or transformed by `llmFactory()`.

The client comes from:

```text
@langchain/openai
```

**---**

**## Inputs & Outputs**

**### Inputs**

This class does not implement its own execution handler.

Input processing is inherited from:

```text
LlmNode
```

The exact input structure cannot be determined from the supplied class alone.

**### Outputs**

Output handling is inherited from `LlmNode`.

The supplied implementation creates the configured `ChatOpenAI` model through `llmFactory()`.

**### Output Example**

The exact output structure depends on the inherited `LlmNode` implementation and is not defined in the supplied code.

**---**

**## Configuration Examples**

**### Basic OpenAI LLM**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "maxRetries": 3
}
```

**### Temperature Configuration**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "temperature": 0.7,
  "maxRetries": 3
}
```

**### System Prompt**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "systemPrompt": "You are a helpful assistant.",
  "temperature": 0.5
}
```

**### Token and Timeout Configuration**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "maxTokens": 1000,
  "timeout": 30000,
  "maxRetries": 3
}
```

<!-- SECTION: examples -->

**## Example Workflow**

```fusion-workflow
src: example.workflow.json
title: Use OpenAI LLM in a workflow
```

<!-- /SECTION: examples -->

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → OpenAI LLM

\- OpenAI LLM → AI processing node

\- Data Processing → OpenAI LLM

\- OpenAI LLM → Function

\- OpenAI LLM → Workflow output

**---**

**## Error Handling**

This class does not implement custom error handling.

Errors during model creation or execution depend on the inherited `LlmNode` behavior and the underlying `ChatOpenAI` implementation.

The schema exposes `maxRetries` with a default value of `3`, but this class itself does not implement a custom retry loop.

**---**

**## Troubleshooting**

**### Authentication Fails**

Verify:

```text
apiKey
```

The schema makes `apiKey` optional, but the remote service may require authentication.

---

**### Model Is Not Available**

Verify:

```text
model
```

Default:

```text
gpt-4o-mini
```

---

**### Requests Time Out**

Configure:

```text
timeout
```

The schema describes this value in milliseconds.

---

**### Requests Fail Repeatedly**

Configure:

```text
maxRetries
```

Default:

```text
3
```

**---**

**## Security**

The node can contain an OpenAI API key.

For production workflows:

\- Store API keys securely

\- Never commit real API keys to Git

\- Do not include real credentials in workflow examples

\- Avoid logging API keys

\- Rotate exposed credentials

**---**

**## Notes**

Metadata label:

```text
OpenAI LLM
```

Metadata description:

```text
A node that uses OpenAI's LLM models.
```

Default model:

```text
gpt-4o-mini
```

Default maximum retries:

```text
3
```

The node does not define its own:

\- Execution handler

\- Input parsing logic

\- Output formatting logic

\- Custom error handling

\- Custom retry loop

The supplied class delegates LLM execution to the inherited `LlmNode`.

The `llmFactory()` method passes the provided parameters directly to `ChatOpenAI`.

The `stop()` method is empty and performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-21` | Initial release |

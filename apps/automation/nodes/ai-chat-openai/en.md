---
node_id: "ai-chat-openai"

title: "AI Chat (OpenAI)"

description: "Connect workflows to OpenAI chat models."

category: "AI / Chat"

version: "1.0.0"

language: "en"

last_updated: "2026-09-18"

author: "Fusion Team"

tags:

- ai-chat

- openai

- llm

- chat

- langchain

related_nodes:

- ai-chat

- ai-chat-openai-like

- ai-chat-lmstudio

---

**# AI Chat (OpenAI)**

> **\*\*Category:\*\*** ai-nodes | **\*\*Type:\*\*** AI Chat Node

Connect workflows to OpenAI chat models using LangChain's `ChatOpenAI` client.

The **\*\*AI Chat (OpenAI)\*\*** node extends the shared `AiChatNode` implementation and provides configuration for the model, API key, temperature, streaming, system message, token limits, penalties, top-p sampling, concurrency, and timeout.

**### Supported Features**

\- Connect to OpenAI chat models

\- Configure the model name

\- Configure an API key

\- Configure temperature

\- Configure streaming

\- Configure a system message

\- Configure maximum tokens

\- Configure maximum completion tokens

\- Configure frequency penalty

\- Configure presence penalty

\- Configure top-p sampling

\- Configure maximum concurrency

\- Configure request timeout

\- Use LangChain `ChatOpenAI`

**### Use Cases**

\- Add OpenAI chat capabilities to workflows

\- Generate text using OpenAI models

\- Configure generation and sampling parameters

\- Control token limits

\- Configure frequency and presence penalties

\- Integrate AI responses with workflow processing

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `model` | `string` | ❌ No | `"gpt-4o-mini"` | OpenAI model name. |
| `apiKey` | `string` | ❌ No | — | API key supplied to `ChatOpenAI`. |
| `temperature` | `number` | ❌ No | — | Sampling temperature. |
| `streaming` | `boolean` | ❌ No | `false` | Streaming configuration exposed by the schema. |
| `systemMessage` | `string` | ❌ No | — | System message configuration exposed by the schema. |
| `maxTokens` | `number` | ❌ No | — | Maximum token configuration. |
| `maxCompletionTokens` | `number` | ❌ No | — | Maximum completion-token configuration. |
| `frequencyPenalty` | `number` | ❌ No | — | Frequency penalty configuration. |
| `presencePenalty` | `number` | ❌ No | — | Presence penalty configuration. |
| `topP` | `number` | ❌ No | — | Top-p sampling value. |
| `maxConcurrency` | `number` | ❌ No | — | Maximum concurrency configuration. |
| `timeout` | `number` | ❌ No | — | Request timeout configuration. |

Default model:

```text
gpt-4o-mini
```

Default streaming:

```text
false
```

**---**

**## Operations**

This class does not define an `operation` field or operation switch.

It extends:

```text
AiChatNode
```

Provider identifier:

```text
openai
```

Chat execution, message processing, and workflow input/output behavior are inherited from the shared `AiChatNode` implementation and are not shown in the supplied code.

**---**

**## Model Construction**

```ts
protected createModel(config: Record<string, unknown>) {
  const { systemMessage, streaming, ...options } = config;
  return new ChatOpenAI(options as any) as any;
}
```

The following values are removed before the remaining configuration is passed to `ChatOpenAI`:

```text
systemMessage
streaming
```

All remaining configuration values are passed through `options`.

The client is instantiated as:

```ts
new ChatOpenAI(options as any)
```

**---**

**## Inputs & Outputs**

**### Inputs**

This class does not implement its own:

```text
handleTick()
```

Input processing is inherited from `AiChatNode`.

**### Outputs**

Output generation and response formatting are also inherited from `AiChatNode`.

The supplied class defines provider metadata, configuration schema, and model construction.

**### Output Example**

The exact output structure cannot be determined from this class alone because execution logic is implemented in the shared `AiChatNode`.

**---**

**## Configuration Examples**

**### Basic OpenAI Configuration**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "streaming": false
}
```

**### Temperature Configuration**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "temperature": 0.7,
  "streaming": false
}
```

**### Token Configuration**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "maxTokens": 1000,
  "maxCompletionTokens": 1000
}
```

**### Sampling and Penalty Configuration**

```json
{
  "model": "gpt-4o-mini",
  "apiKey": "YOUR_API_KEY",
  "temperature": 0.5,
  "topP": 0.9,
  "frequencyPenalty": 0.2,
  "presencePenalty": 0.1
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → AI Chat (OpenAI)

\- HTTP Request → AI Chat (OpenAI)

\- AI Chat (OpenAI) → Function

\- AI Chat (OpenAI) → Notification

\- Data Processing → AI Chat (OpenAI) → Output

**---**

**## Error Handling**

This class does not implement custom error handling.

Errors produced during model creation or invocation depend on the inherited `AiChatNode` behavior and the underlying `ChatOpenAI` implementation.

Because the shared `AiChatNode` source was not supplied, this documentation does not assume a specific inherited error format.

**---**

**## Troubleshooting**

**### Authentication Fails**

Verify:

```text
apiKey
```

The schema makes `apiKey` optional, but the OpenAI service may require authentication for requests.

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

**### systemMessage or streaming Behavior**

`systemMessage` and `streaming` exist in the schema, but `createModel()` removes them before passing the remaining options to `ChatOpenAI`.

Their complete behavior depends on the inherited `AiChatNode` implementation.

---

**### Token Configuration**

The schema exposes both:

```text
maxTokens
maxCompletionTokens
```

Both remain in `options` when configured because `createModel()` only removes `systemMessage` and `streaming`.

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
AI Chat (OpenAI)
```

Metadata description:

```text
OpenAI chat models.
```

Provider identifier:

```text
openai
```

Default model:

```text
gpt-4o-mini
```

Default streaming:

```text
false
```

The node does not define its own:

\- `handleTick()`

\- Input parsing logic

\- Output formatting logic

\- Custom error handling

\- Custom API base URL

The supplied class delegates chat execution to the shared `AiChatNode`.

The `createModel()` method removes `systemMessage` and `streaming` before passing the remaining configuration to `ChatOpenAI`.

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-18` | Initial release |

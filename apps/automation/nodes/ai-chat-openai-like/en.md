---
node_id: "ai-chat-openai-like"

title: "AI Chat (OpenAI-compatible)"

description: "Connect to any OpenAI-compatible chat-completions endpoint."

category: "AI / Chat"

version: "1.0.0"

language: "en"

last_updated: "2026-09-18"

author: "Fusion Team"

tags:

- ai-chat

- openai-compatible

- llm

- chat-completions

- langchain

related_nodes:

- ai-chat

- ai-chat-lmstudio

- function

---

**# AI Chat (OpenAI-compatible)**

> **\*\*Category:\*\*** ai-nodes | **\*\*Type:\*\*** AI Chat Node

Connect workflows to any OpenAI-compatible chat-completions endpoint using LangChain's `ChatOpenAI` client.

The **\*\*AI Chat (OpenAI-compatible)\*\*** node extends the shared `AiChatNode` implementation and allows the model, API key, base URL, temperature, streaming setting, system message, token limit, sampling configuration, concurrency, and timeout to be configured.

**### Supported Features**

\- Connect to OpenAI-compatible endpoints

\- Configure the model name

\- Configure a custom API key

\- Configure a custom base URL

\- Configure temperature

\- Configure streaming

\- Configure a system message

\- Configure maximum output tokens

\- Configure top-p sampling

\- Configure maximum concurrency

\- Configure request timeout

\- Use LangChain `ChatOpenAI`

**### Use Cases**

\- Connect workflows to OpenAI-compatible LLM providers

\- Use a custom OpenAI-compatible API endpoint

\- Switch between compatible models

\- Configure generation parameters

\- Use self-hosted or third-party compatible services

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `model` | `string` | ❌ No | `"gpt-4.1-mini"` | Model name. |
| `apiKey` | `string` | ❌ No | — | API key supplied to `ChatOpenAI`. |
| `baseUrl` | `string` | ❌ No | `"https://api.openai.com/v1"` | OpenAI-compatible API base URL. |
| `temperature` | `number` | ❌ No | — | Sampling temperature. |
| `streaming` | `boolean` | ❌ No | `false` | Streaming configuration exposed by the schema. |
| `systemMessage` | `string` | ❌ No | — | System message configuration exposed by the schema. |
| `maxTokens` | `number` | ❌ No | — | Maximum generated tokens. |
| `topP` | `number` | ❌ No | — | Top-p sampling value. |
| `maxConcurrency` | `number` | ❌ No | — | Maximum concurrency configuration. |
| `timeout` | `number` | ❌ No | — | Request timeout configuration. |

Default model:

```text
gpt-4.1-mini
```

Default base URL:

```text
https://api.openai.com/v1
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

Provider:

```text
openai-like
```

Chat execution, message handling, and workflow input/output behavior are inherited from the shared `AiChatNode` implementation and are not shown in the supplied code.

**---**

**## Model Construction**

The model is created using:

```ts
protected createModel(config: Record<string, unknown>) {
  const { baseUrl, systemMessage, streaming, ...options } = config;
  return new ChatOpenAI({
    ...options,
    configuration: { baseURL: baseUrl as string }
  } as any) as any;
}
```

The following values are removed before `options` is passed to `ChatOpenAI`:

```text
baseUrl
systemMessage
streaming
```

The remaining configuration is passed through:

```text
...options
```

The endpoint is configured using:

```ts
configuration: {
  baseURL: baseUrl as string
}
```

**---**

**## Inputs & Outputs**

**### Inputs**

This class does not implement its own:

```text
handleTick()
```

Input handling is inherited from `AiChatNode`.

**### Outputs**

Output generation and formatting are also inherited from `AiChatNode`.

The supplied class only defines provider metadata, configuration schema, and model construction.

**### Output Example**

The exact output structure cannot be determined from this class alone because the execution logic is implemented in the shared `AiChatNode`.

**---**

**## Configuration Examples**

**### Default Endpoint**

```json
{
  "model": "gpt-4.1-mini",
  "apiKey": "YOUR_API_KEY",
  "baseUrl": "https://api.openai.com/v1",
  "streaming": false
}
```

**### Custom OpenAI-Compatible Endpoint**

```json
{
  "model": "YOUR_MODEL_NAME",
  "apiKey": "YOUR_API_KEY",
  "baseUrl": "https://your-compatible-api.example/v1",
  "temperature": 0.7,
  "streaming": false
}
```

**### Generation Configuration**

```json
{
  "model": "YOUR_MODEL_NAME",
  "apiKey": "YOUR_API_KEY",
  "baseUrl": "https://your-compatible-api.example/v1",
  "temperature": 0.5,
  "maxTokens": 1000,
  "topP": 0.9,
  "timeout": 30000
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → AI Chat (OpenAI-compatible)

\- HTTP Request → AI Chat (OpenAI-compatible)

\- AI Chat (OpenAI-compatible) → Function

\- AI Chat (OpenAI-compatible) → Notification

\- Data Processing → AI Chat (OpenAI-compatible) → Output

**---**

**## Error Handling**

This class does not implement custom error handling.

Errors during model creation or invocation depend on the inherited `AiChatNode` behavior and the underlying `ChatOpenAI` implementation.

Because the shared `AiChatNode` source was not supplied, no specific inherited error format is assumed.

**---**

**## Troubleshooting**

**### Endpoint Cannot Be Reached**

Verify:

```text
baseUrl
```

Default:

```text
https://api.openai.com/v1
```

---

**### Authentication Fails**

Verify:

```text
apiKey
```

The schema makes `apiKey` optional, but the configured remote endpoint may require authentication.

---

**### Model Is Not Found**

Verify:

```text
model
```

The configured model must exist and be accessible through the selected endpoint.

Default:

```text
gpt-4.1-mini
```

---

**### systemMessage or streaming Behavior**

`systemMessage` and `streaming` exist in the schema, but `createModel()` removes them before passing the remaining options to `ChatOpenAI`.

Their complete behavior depends on the inherited `AiChatNode` implementation.

**---**

**## Security**

The node can contain an API key for the configured AI endpoint.

For production workflows:

\- Store API keys securely

\- Never commit real API keys to Git

\- Do not include real credentials in workflow examples

\- Avoid logging authentication information

\- Rotate exposed credentials

\- Use trusted OpenAI-compatible endpoints

**---**

**## Notes**

Metadata label:

```text
AI Chat (OpenAI-compatible)
```

Metadata description:

```text
Any OpenAI-compatible chat-completions endpoint.
```

Provider identifier:

```text
openai-like
```

Default model:

```text
gpt-4.1-mini
```

Default base URL:

```text
https://api.openai.com/v1
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

The supplied class delegates chat execution to the shared `AiChatNode`.

The `createModel()` method removes `baseUrl`, `systemMessage`, and `streaming` before passing the remaining options to `ChatOpenAI`.

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-18` | Initial release |

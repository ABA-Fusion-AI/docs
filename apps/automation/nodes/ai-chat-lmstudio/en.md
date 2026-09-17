---
author: Fusion Team
category: AI / Chat Models
description: Run locally served LM Studio models through an
  OpenAI-compatible API.
language: en
last_updated: 2026-09-17
node_id: ai-chat-lmstudio
related_nodes:
- ai-chat
- openai
tags:
- ai
- lm-studio
- llm
- local-ai
- openai-compatible
- langchain
title: AI Chat (LM Studio)
version: 1.0.0
---

**\# AI Chat (LM Studio)**

> **\*\*Category:\*\*** ai-nodes \| **\*\*Type:\*\*** AI Chat Node

Connect workflows to **\*\*LM Studio\*\*** models served locally through
an OpenAI-compatible API.

The node extends the shared `AiChatNode` implementation and creates a
LangChain `ChatOpenAI` client configured with the LM Studio base URL.

**\### Supported Features**

\- Connect to locally served LM Studio models

\- Use an OpenAI-compatible endpoint

\- Configure model name and API key

\- Configure a custom base URL

\- Configure temperature

\- Expose streaming and system-message settings

\- Configure maximum tokens, top-p, concurrency, and timeout

\- Create the model through LangChain `ChatOpenAI`

**\### Use Cases**

\- Run local LLMs in Fusion workflows

\- Connect workflows to LM Studio

\- Use different locally available models

\- Use a custom OpenAI-compatible LM Studio endpoint

**---**

**\## Configuration**

**\### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `model` | `string` | ❌ No | `"llama3"` | Model identifier. |
| `apiKey` | `string` | ❌ No | `"lm-studio"` | API key value passed to the client. |
| `baseUrl` | `string` | ❌ No | `"http://localhost:1234/v1"` | LM Studio OpenAI-compatible API base URL. |
| `temperature` | `number` | ❌ No | — | Model temperature. |
| `streaming` | `boolean` | ❌ No | `false` | Streaming setting exposed by the node. |
| `systemMessage` | `string` | ❌ No | — | Optional system-message configuration. |
| `maxTokens` | `number` | ❌ No | — | Maximum-token configuration. |
| `topP` | `number` | ❌ No | — | Top-p sampling configuration. |
| `maxConcurrency` | `number` | ❌ No | — | Maximum concurrency configuration. |
| `timeout` | `number` | ❌ No | — | Timeout configuration. |

**\### Default Configuration**

``` json
{
  "model": "llama3",
  "apiKey": "lm-studio",
  "baseUrl": "http://localhost:1234/v1",
  "streaming": false
}
```

**---**

**\## Operations**

The node does not expose an `operation` parameter.

Its provider identifier is:

``` ts
protected readonly provider = "lmstudio";
```

Chat execution is inherited from:

``` ts
AiChatNode
```

**---**

**\## Request / Model Construction**

The node creates a LangChain model using:

``` ts
new ChatOpenAI(...)
```

Configuration is destructured as:

``` ts
const { baseUrl, systemMessage, streaming, ...options } = config;
```

The model is then created as:

``` ts
new ChatOpenAI({
  ...options,
  configuration: {
    baseURL: baseUrl as string
  }
})
```

Therefore `baseUrl` is mapped to:

``` text
configuration.baseURL
```

The remaining `options` are passed directly into `ChatOpenAI`.

`systemMessage` and `streaming` are removed from the object before
`...options` is passed to the constructor.

**---**

**\## Inputs & Outputs**

The supplied class does not implement `handleTick()`.

Input processing, chat invocation, and output formatting are inherited
from:

``` ts
AiChatNode
```

Therefore the exact workflow input and output structure cannot be
determined from this implementation alone.

**---**

**\## Configuration Examples**

**\### Default Local LM Studio**

``` json
{
  "model": "llama3",
  "apiKey": "lm-studio",
  "baseUrl": "http://localhost:1234/v1",
  "streaming": false
}
```

**\### Custom Local Model**

``` json
{
  "model": "local-model-name",
  "apiKey": "lm-studio",
  "baseUrl": "http://localhost:1234/v1",
  "temperature": 0.7,
  "maxTokens": 1000,
  "topP": 0.9
}
```

**\### Custom LM Studio Server**

``` json
{
  "model": "local-model-name",
  "apiKey": "lm-studio",
  "baseUrl": "http://192.168.1.10:1234/v1",
  "timeout": 60000,
  "maxConcurrency": 2
}
```

**---**

**\## Workflow Integration**

**\### Common Patterns**

\- Prompt/Input → AI Chat (LM Studio)

\- Trigger → AI Chat (LM Studio) → Output Processing

\- AI Chat (LM Studio) → Function

\- AI Chat (LM Studio) → Database

\- AI Chat (LM Studio) → Notification

The exact chat-flow behavior depends on the shared `AiChatNode`.

**---**

**\## Error Handling**

The supplied implementation does not define LM Studio-specific error
handling.

Errors during model creation, connection, or execution are handled by
the inherited AI Chat implementation or the underlying LangChain/OpenAI
client.

**---**

**\## Troubleshooting**

**\### Cannot Connect to LM Studio**

Check the configured base URL.

Default:

``` text
http://localhost:1234/v1
```

`localhost` refers to the environment where this node executes.

------------------------------------------------------------------------

**\### Model Is Not Available**

The default model value is:

``` text
llama3
```

The node does not verify that this model is loaded in LM Studio.
Configure `model` with an identifier available from the running server.

------------------------------------------------------------------------

**\### Streaming Behavior**

The schema exposes `streaming`, but `createModel()` extracts it before
spreading `options` into `ChatOpenAI`.

The supplied method therefore does not directly pass `streaming` to the
constructor.

------------------------------------------------------------------------

**\### System Message Behavior**

`systemMessage` is also extracted before model construction.

Its actual use depends on the inherited `AiChatNode` implementation.

**---**

**\## Security**

The default configuration is intended for a local LM Studio endpoint.

For remotely exposed servers:

\- Protect the inference endpoint from unauthorized access

\- Use appropriate credentials when required

\- Avoid committing real credentials to Git

\- Use protected configuration for sensitive values

\- Prefer encrypted transport for remote endpoints

**---**

**\## Notes**

Metadata label:

``` text
AI Chat (LM Studio)
```

Metadata description:

``` text
Locally served LM Studio models through its OpenAI-compatible API.
```

Provider:

``` text
lmstudio
```

Defaults:

``` text
model = llama3
apiKey = lm-studio
baseUrl = http://localhost:1234/v1
streaming = false
```

The node extends:

``` text
AiChatNode
```

The client implementation is:

``` text
ChatOpenAI
```

The supplied class does not directly define:

\- `handleTick()`

\- Prompt construction

\- Chat-history handling

\- Output formatting

\- LM Studio server startup

\- Model downloading or loading

\- Custom retry logic

\- LM Studio-specific error transformation

These behaviors are inherited, handled by dependencies, or outside the
supplied implementation.

**---**

**\## Changelog**

|Version |  Date  |  Changes|
|--- |--- |---|
|1.0.0 |  2026-09-17 |  Initial release|

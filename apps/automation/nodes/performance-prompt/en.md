---
node_id: "performance-prompt"
title: "Performance Prompt"
description: "Fetch and compile a deployed prompt from Performance."
category: "Prompt Management"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:

- performance
- langfuse
- prompt-management
- prompt-templates
- llm
- prompt-compilation
- prompt-versioning

related_nodes:
- function
- if
- llm

---

# Performance Prompt

> **Category:** prompt-management-nodes | **Type:** Action Node

Fetch a **deployed prompt** from Performance (backed by Langfuse) and **compile** it with runtime variables.

The **Performance Prompt** node retrieves a named prompt — either the current production version or a specific version number — from the connected Langfuse-backed prompt store, substitutes the provided `variables` into the template, and returns the compiled prompt.

### Supported Features

- Fetch a prompt by name from Performance/Langfuse
- Retrieve either the current **Production** version or a **specific version number**
- Compile the prompt template with key-value `variables`
- Conditional parameter display: `promptVersionNumber` only shown when `promptVersion` is `"Specific Version"`
- Automatic flush of the Langfuse client after each fetch, ensuring telemetry/usage is sent

### Use Cases

- Centrally manage and version LLM prompts outside the workflow definition
- Fetch the latest production prompt for a chatbot or agent step
- Pin a workflow to a specific, tested prompt version for reproducibility
- Inject dynamic variables (user name, context, retrieved data) into a shared prompt template
- Feed the compiled prompt directly into an `LLM` node

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `promptName` | `string` | ✅ Yes | — | Name of the deployed prompt in Langfuse. |
| `promptVersion` | `enum` | ❌ No | `"Production"` | Which version to fetch: `Production` or `Specific Version`. |
| `promptVersionNumber` | `number` | ✅ Yes (if `promptVersion = "Specific Version"`) | — | Specific version number to fetch. |
| `variables` | `object` (string key/value map) | ❌ No | — | Key-value pairs substituted into the prompt template. |

`promptVersionNumber` is only shown/used when `promptVersion` is set to `"Specific Version"`; for `"Production"`, the node fetches the currently deployed production version.

---

## How It Works

1. A Langfuse client is built from the node's `workflowContext` (connection/credentials are resolved from the workflow environment, not from node parameters).
2. `client.getPrompt(promptName, promptVersionNumber)` fetches the prompt — passing `undefined` for the version when `promptVersionNumber` is not set, which resolves to the production version.
3. `prompt.compile(variables)` substitutes the provided variables into the prompt template.
4. `client.flushAsync()` is called to ensure any pending Langfuse telemetry is sent before the node returns.
5. The compiled prompt is returned directly as the node's output.

---

## Inputs & Outputs

### Inputs

The node does not use workflow input data — all configuration comes from `promptName`, `promptVersion`, `promptVersionNumber`, and `variables`.

### Outputs

The node returns the **compiled prompt** exactly as returned by `prompt.compile(variables)` — its shape depends on the Langfuse prompt type (a text string for text prompts, or a structured message array for chat prompts).

---

## Output Example

### Text Prompt

```text
You are a helpful assistant for Acme Corp. The customer's name is Julien and their order number is 48213.
```

### Chat Prompt

```json
[
  { "role": "system", "content": "You are a helpful assistant for Acme Corp." },
  { "role": "user", "content": "My order number is 48213, can you help?" }
]
```

The exact output shape depends on how the prompt named `promptName` was authored in Performance/Langfuse.

---

## Configuration Examples

### Fetch Production Prompt

```json
{
  "promptName": "customer-support-greeting",
  "promptVersion": "Production",
  "variables": {
    "customerName": "Julien",
    "orderNumber": "48213"
  }
}
```

### Fetch a Specific Version

```json
{
  "promptName": "customer-support-greeting",
  "promptVersion": "Specific Version",
  "promptVersionNumber": 3,
  "variables": {
    "customerName": "Julien",
    "orderNumber": "48213"
  }
}
```

### Prompt Without Variables

```json
{
  "promptName": "static-welcome-message"
}
```

---

## Workflow Integration

### Sample Workflow: Performance Prompt → LLM

```json
{
  "nodes": [
    {
      "id": "fetch-prompt",
      "type": "performance-prompt",
      "config": {
        "promptName": "customer-support-greeting",
        "variables": {
          "customerName": "Julien"
        }
      }
    },
    {
      "id": "run-llm",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: Function (build variables) → Performance Prompt → LLM

```json
{
  "nodes": [
    {
      "id": "build-variables",
      "type": "function"
    },
    {
      "id": "fetch-prompt",
      "type": "performance-prompt",
      "config": {
        "promptName": "order-status-update",
        "promptVersion": "Specific Version",
        "promptVersionNumber": 2
      }
    },
    {
      "id": "run-llm",
      "type": "llm"
    }
  ]
}
```

### Common Patterns

- Function (assemble `variables` from workflow data) → Performance Prompt → LLM — dynamic prompt injection
- Performance Prompt (`Specific Version`) → LLM — pin a tested prompt version for a production workflow
- Performance Prompt (`Production`) → LLM — always use the latest deployed prompt without redeploying the workflow

---

## Error Handling

The node does not define custom error messages — errors propagate directly from the underlying Langfuse client and prompt compilation calls. Common failure sources include:

- **Prompt not found** — `client.getPrompt` fails if no prompt named `promptName` exists in the connected Langfuse project.
- **Version not found** — fetching a `promptVersionNumber` that does not exist for the given `promptName`.
- **Compilation error** — `prompt.compile` may throw or produce unexpected output if `variables` does not supply all placeholders the template expects.
- **Connection/auth error** — `buildLangfuseClient` may fail if the workflow's Langfuse connection is not configured or credentials are invalid.

---

## Troubleshooting

### Prompt Not Found

**Cause**

`promptName` does not match any prompt deployed in the connected Langfuse project, or the prompt exists in a different Langfuse project/environment than the one configured for this workflow.

**Solution**

Verify the exact prompt name in the Performance/Langfuse dashboard, and confirm the workflow is connected to the correct project.

---

### Specific Version Not Found

**Cause**

`promptVersionNumber` does not correspond to an existing version of `promptName`.

**Solution**

Check the prompt's version history in Langfuse and use a valid version number, or switch `promptVersion` back to `"Production"`.

---

### Placeholders Left Unfilled in Compiled Output

**Cause**

The prompt template references variables that were not included in `variables`.

**Solution**

Compare the template's placeholders (visible in the Langfuse prompt editor) against the keys provided in `variables`, and add any missing ones.

---

### Connection or Authentication Failure

**Cause**

The workflow's Langfuse connection is missing, misconfigured, or using invalid/expired credentials.

**Solution**

Check the Langfuse connection configured for this workflow environment (outside this node's own parameters).

---

## Security

The node relies on the workflow's configured Langfuse connection (via `buildLangfuseClient(this.workflowContext)`) — no credentials are entered directly on this node.

`client.flushAsync()` ensures prompt-fetch telemetry is sent to Langfuse synchronously before the node completes, rather than being buffered indefinitely.

---

## Notes

The node returns the compiled prompt directly with no additional wrapping or metadata (no `success`, `promptName`, or version echoed back in the output).

The node does not:

- List available prompts or versions
- Cache fetched prompts between ticks
- Validate that all template variables were supplied before compiling
- Support prompt creation or editing (fetch/compile only)
- Send the compiled prompt to an LLM itself — that is left to a downstream `LLM` node

It is intended to be a fetch-and-compile step that decouples prompt content and versioning from the workflow definition.

---

## Related

- [LLM](./llm.md) – Send the compiled prompt to a language model
- [Function](./function.md) – Build or transform the `variables` object before compilation
- [If](./if.md) – Route workflows based on prompt content or fetch success

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-17 | Initial release |
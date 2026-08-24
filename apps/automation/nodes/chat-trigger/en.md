---
  node_id: "chat-trigger"
  title: "Chat Trigger"
  description: "Starts the workflow when someone sends a message in the chat box, passing the typed text into the graph."
  category: "triggers"
  subcategory: "manual-scheduled"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-08-24"
  author: "Fusion Team"
  tags:
    - chat
    - trigger
    - conversation
    - manual
    - no-code
  related_nodes:
    - chat-respond
    - manual-trigger
    - tool-input
---

<!-- SECTION: overview -->
  # Chat Trigger

  > **Category:** Triggers &amp; Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Trigger Node

  Starts the workflow when someone types a message in the chat box, and passes what they typed into the graph.

  Pair it with a **Chat Response** node at the other end. Without one, a message starts the workflow but nothing ever comes back to the chat box.

### Use Cases

- Try a prompt against an LLM node without leaving the builder
- Give a workflow a conversational front end for testing
- Drive a support or triage flow from typed questions
- Check what an agent actually replies before publishing it as a tool

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `greeting` | `string` | ❌ No | — | Shown in the chat box before the first message. Leave empty for no greeting. |
| `placeholder` | `string` | ❌ No | `Type a message…` | Placeholder text in the chat box's input field. |

Both are presentation only — they change how the chat box looks, not what the workflow receives.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs &amp; Outputs

### Inputs

None. This is a trigger — it starts the workflow rather than receiving from it.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when a message arrives |
| `error` | `Error` | Emitted when the chat capability is unavailable |

### Output Schema (`success`)

```json
{
  "text": "What is the status of order 1024?"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `text` | `string` | Exactly what the user typed. Empty string if they sent nothing. |

### Session

The node puts a `chatRequestId` on the session, which travels unchanged through
every downstream node. **Chat Response** reads it to work out which message it
is answering. You never need to handle it yourself, but it is why replies stay
matched to their questions when several are in flight.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Example

A workflow that answers questions with an LLM:

```
Chat Trigger  →  Agent / LLM  →  Chat Response
```

1. Add a **Chat Trigger**. Optionally set a greeting.
2. Wire its `success` output into your LLM or agent node, referencing the message with an expression such as `{{ $node["Chat Trigger"].text }}`.
3. Wire that node into a **Chat Response**.
4. **Start the workflow**, then open the chat box from the toolbar and type.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| The chat button is missing from the toolbar | The graph has no Chat Trigger node | Add one; the button appears only for workflows that can be chatted with |
| "Start the workflow to send messages" | The workflow is not running | Trigger nodes register when the workflow starts, not when it is saved. Press start. |
| "Add a Chat Response node — without one nothing can reply" | The graph has no responder | Add a **Chat Response** and wire your logic into it |
| Messages send but no reply arrives | Nothing reaches the Chat Response node | Check the path from trigger to responder. A branch that ends early leaves the chat box waiting. |
| The reply is `[object Object]`-ish JSON | The upstream node emitted an object with no recognisable text field | Set the Chat Response node's `message` parameter to the field you want, e.g. `{{ $node["Agent"].output }}` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- **[Chat Response](../chat-respond/en.md)** — the other end; sends the reply back
- **[Manual Trigger](../manual-trigger/en.md)** — starts a workflow with no payload
- **[Tool Input](../tool-input/en.md)** — the equivalent entry point for AI agents calling over MCP

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-08-24 | Initial release |

<!-- /SECTION: changelog -->

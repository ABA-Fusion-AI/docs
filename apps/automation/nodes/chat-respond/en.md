---
  node_id: "chat-respond"
  title: "Chat Response"
  description: "Sends a reply back to the chat box. Whatever reaches this node is what the user sees as the answer."
  category: "triggers"
  subcategory: "manual-scheduled"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-08-24"
  author: "Fusion Team"
  tags:
    - chat
    - response
    - conversation
    - reply
    - no-code
  related_nodes:
    - chat-trigger
    - tool-output
    - respond-to-webhook
---

<!-- SECTION: overview -->
  # Chat Response

  > **Category:** Triggers &amp; Ingress&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

  Sends a reply back to the chat box. Whatever reaches this node is what appears as the answer to the message that started the run.

  This is the other half of **Chat Trigger**. A workflow with a trigger but no response node runs fine — the chat box simply never hears back.

### Use Cases

- Return an LLM's answer to the person who asked
- Send a formatted summary rather than the raw output of the previous node
- Reply with a fixed acknowledgement while other branches keep working

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `message` | `any` | ❌ No | — | What to say back. Supports expressions. Leave empty to send whatever reached this node. |

### How the reply is rendered

The chat box shows text, so the value is reduced to text before display:

- a **string** is shown as-is
- a **number** or **boolean** is shown as its text form
- an **object** is searched for a `text`, `message`, `content`, `output` or `response` field, and that is shown
- anything else is shown as formatted JSON

Set `message` explicitly when you want a specific field rather than relying on that search — it is a convenience for the common shapes, not a guarantee.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs &amp; Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | The reply, or the data to derive it from |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The reply that was sent |
| `error` | `Error` | Emitted when the reply could not be produced |

### Output Schema (`success`)

```json
{
  "chatRequestId": "6f1c…",
  "text": "Order 1024 shipped on Tuesday.",
  "raw": { "answer": "Order 1024 shipped on Tuesday.", "confidence": 0.92 }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `chatRequestId` | `string` | Which message this answers. Read from the session that Chat Trigger set. |
| `text` | `string` | The rendered reply, as the chat box shows it |
| `raw` | `any` | The unreduced value, for anything wired downstream that wants the real shape |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Example

```
Chat Trigger  →  Agent / LLM  →  Chat Response
```

To reply with one field of an object rather than the whole thing, set `message`
to an expression such as `{{ $node["Agent"].answer }}`.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| No reply reaches the chat box | Nothing reaches this node | Follow the path from the trigger; a filter or a branch that ends early leaves the box waiting |
| The reply is JSON rather than a sentence | The upstream output has no recognisable text field | Set `message` to the field you want |
| Two replies arrive for one message | Two branches both reach a response node | Only the first correlated reply is shown; merge the branches if the duplication is not intended |
| The reply appears in the logs but not the chat box | The run was not started from the chat box | A run started another way carries no `chatRequestId`, so there is nobody waiting to show it to |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- **[Chat Trigger](../chat-trigger/en.md)** — the other end; starts the run
- **[Tool Output](../tool-output/en.md)** — the equivalent for AI agents calling over MCP
- **[Respond to Webhook](../respond-to-webhook/en.md)** — the equivalent for HTTP callers

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-08-24 | Initial release |

<!-- /SECTION: changelog -->

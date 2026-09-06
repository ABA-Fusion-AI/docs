---
  node_id: "agent-v2"
  title: "Agent V2"
  description: "An AI agent with composable memory, queued turns and bounded iterations. Conversation history, execution checkpoints and long-term recall are supplied by a connected Memory node rather than held in the worker."
  category: "generative-ai"
  subcategory: "agents-chat"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-08-28"
  author: "Fusion Team"
  tags:
    - agent
    - ai
    - llm
    - memory
    - conversation
  related_nodes:
    - agent
    - agentic-flow
---

<!-- SECTION: overview -->
  # Agent V2

  > **Category:** Generative AI &amp; LLMs&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Agent Node

  An AI agent that answers a user's message using a connected language model.

  Agent V2 ships **alongside** the original Agent, which is unchanged. Existing
  workflows keep working exactly as they did; nothing is migrated and nothing
  needs to be. Choose V2 for new work when you want conversation history that
  survives a restart, or turns that queue instead of colliding.

### What is different from Agent

| | Agent | Agent V2 |
|---|---|---|
| Conversation identity | A **Thread ID** you type into the node | Derived per conversation at run time |
| Memory | Held inside the worker process | Supplied by a connected **Memory** node |
| Survives a restart | No | Only when the connected memory says it is durable |
| A second message mid-turn | Undefined | Queued, then run in order |
| Iteration limit | None | **Max Iterations**, enforced per turn |

The **Thread ID** field is gone on purpose. In Agent, that single value tied a
whole workflow to one conversation for its entire life, so every user shared
one history. Agent V2 works out conversation identity when the turn runs.

### Use Cases

- A support assistant whose history survives a deployment
- A chat workflow where a user may send a follow-up before the first reply lands
- An agent whose cost per turn needs a hard ceiling

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
  ## Configuration

| Field | Required | Default | Description |
|---|---|---|---|
| **System Prompt** | No | — | Instructions given to the agent at the start of every turn. |
| **Max Iterations** | Yes | `10` | Upper bound on model and tool round trips within a single turn. |

### System Prompt

Sent ahead of the user's message on every turn. Leave it empty and no system
message is sent at all, rather than an empty one.

### Max Iterations

A ceiling on the round trips one turn may make, so a model that keeps calling
itself cannot run up an unbounded bill. Reaching the limit ends the turn on the
**Error** output.

### Concurrent turns are queued, and this is not adjustable

A message that arrives while a turn is still running waits, then runs in order,
so replies always match the order the messages were sent in. There is no setting
for it: queueing is the only behaviour this node has, and a field offering one
option would suggest a choice that does not exist.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
  ## Inputs and Outputs

### Connections

| Handle | Direction | Required | Description |
|---|---|---|---|
| **LLM** | In | **Yes** | The language model that answers. Connect exactly one. |
| **Memory** | In | No | Supplies conversation history, checkpoints, or long-term recall. Connect at most one. |
| **Tool** | In | No | Drawn, but not yet executable — connecting one fails the turn. See below. |
| **Input** | In | Yes | The user's message. |
| **Success** | Out | — | The agent's reply, as text. |
| **Error** | Out | — | A typed failure. |

### Running without Memory

Valid, and sometimes what you want. With no Memory node connected the agent is
**single-turn**: every message is answered on its own with no history. Use it
for one-shot classification or summarising, where carrying history would only
add cost.

### Durability is declared, not guessed

A connected Memory node states whether it is durable. The builder shows that
directly, because a memory that is *not* durable loses its history when the
worker restarts or the workflow moves between workers. That is fine while you
are building and rarely fine in production.

### Attachments

Agent V2 is **text-only** in this release. Sending an image or a PDF returns a
typed `AGENT_CAPABILITY_UNSUPPORTED` error on the **Error** output rather than
passing a description of the file to the model. This is deliberate: silently
sending the model a JSON blob about a file produces a confident answer about
something it never saw.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: workflow-example -->
  ## Example

A support assistant that remembers the conversation.

```fusion-workflow
src: example.workflow.json
title: Support assistant with memory
```

1. **Chat Trigger** receives the user's message.
2. **OpenAI LLM** is connected to the agent's **LLM** handle.
3. **Memory** is connected to the **Memory** handle, so the agent can see the
   conversation so far.
4. **Agent V2** answers on **Success**, and **Chat Response** sends that
   answer back to the chat box.

Configuration:

- **System Prompt:** `You are a support assistant. Be concise. If you do not know, say so.`
- **Max Iterations:** `10`

Remove the Memory connection and the same workflow still runs — each message is
just answered independently.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
  ## Troubleshooting

### "Agent V2 requires a connected LLM node"

Nothing is connected to the **LLM** handle, or the connected node is not a
language model. Connect exactly one LLM node.

### The reply forgets the previous message

No Memory node is connected, so the agent is single-turn by design. Connect a
Memory node to keep history.

### History disappears after a restart

The connected memory is not durable. The builder marks it **Memory is not
durable**. Volatile memory is fine while developing; switch to a durable one
before relying on it.

### "No transcript authority"

The connected memory provides checkpoints but no conversation history, so runs
would work while the chat showed nothing. Connect a memory that supplies the
transcript.

### `AGENT_CAPABILITY_UNSUPPORTED` on an attachment

Expected. This release is text-only — see **Attachments** above.

### "Agent V2 does not support tools yet"

A Tool node is wired to this agent. The socket **is** drawn on the canvas — it
is there ahead of the runtime that will use it — but tool execution has not
landed yet. Running anyway would answer with the tools silently missing, so the
node fails the turn instead of quietly ignoring them. Disconnect the tool, or
use **Agent** (V1), which supports tools today.

### The turn ends without a reply

The turn hit **Max Iterations**. Raise the limit, or simplify the prompt so the
agent needs fewer round trips.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
  ## Related Nodes

- **Agent** — the original agent. Unchanged, still supported, still the right
  choice for existing workflows.
- **Agentic Flow** — multi-step orchestration.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
  ## Changelog

### 1.0.0

- Initial release: text-only, one LLM, optional composable Memory.
- Queued turns, bounded iterations, cancellable runs.
- Tools, attachments and structured output arrive in later releases.

<!-- /SECTION: changelog -->

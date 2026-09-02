---
  node_id: "mongodb-memory"
  title: "MongoDB Memory"
  description: "Durable conversation memory, execution checkpoints and long-term recall in your own MongoDB, with per-collection naming."
  category: "generative-ai"
  subcategory: "memory"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-09-02"
  author: "Fusion Team"
  tags:
    - memory
    - mongodb
    - agent
    - conversation
    - checkpoints
  related_nodes:
    - agent-v2
    - memory
    - postgres-memory
---

<!-- SECTION: overview -->
  # MongoDB Memory

  > **Category:** Generative AI &amp; LLMs&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Memory Node

  Conversation memory in your own MongoDB. Connect it to an Agent V2 node's
  **Memory** handle.

  Durable and reachable from every worker. Collection names are configurable,
  so several environments can share one database without colliding.

### What it provides

| Capability | Provided | What it means |
|---|---|---|
| Conversation | Yes | The transcript the chat displays. |
| Checkpoints | Yes | An interrupted run resumes where it stopped. |
| Long-term recall | Yes | Values kept across separate conversations. |

### Use Cases

- A deployment already standardised on MongoDB
- Several environments sharing one cluster, separated by collection
- Conversations that should expire after a fixed period of inactivity

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
  ## Configuration

| Field | Required | Default | Description |
|---|---|---|---|
| **Connection URI** | Yes | — | `mongodb://user:password@host:27017`. |
| **Database** | No | from the URI | Leave empty to use the database named in the URI. |
| **Checkpoint Collection** | No | driver default | Where checkpoints are stored. |
| **Checkpoint Writes Collection** | No | driver default | Where in-flight checkpoint writes are stored. |
| **Long-term Collection** | No | driver default | Must differ from the checkpoint collections. |
| **TTL (seconds)** | No | — | Leave empty to keep checkpoints until you delete them. |

### The long-term collection must differ from the checkpoint collections

They hold different shapes of document. Pointing two of these fields at one
collection lets a long-term item be read back where a checkpoint was expected,
which fails in ways that are hard to trace back here. Give each its own name,
or leave all three empty and take the defaults.

### Retention

**TTL (seconds)** is enforced by a MongoDB TTL index on the checkpoints, keyed
on last update — so an idle conversation expires and an active one does not.
The index is created when the node starts.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
  ## Inputs and Outputs

A memory node has **no data inputs or outputs**. It is a capability you attach
to an agent, not a step in a chain.

| Handle | Direction | Required | Description |
|---|---|---|---|
| **Memory** | Out | — | Connect to an Agent V2 node's **Memory** handle. |

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: workflow-example -->
  ## Example

```mermaid
flowchart LR
  trigger([Chat Trigger]) --> agent[Agent V2]
  agent --> reply([Send Reply])
  llm[OpenAI LLM] -- llm --> agent
  mem[(MongoDB Memory)] -- memory --> agent
```

Configuration:

- **Connection URI:** `mongodb://fusion:••••@mongo:27017`
- **Database:** `fusion_agents`
- **Long-term Collection:** `agent_items`
- **TTL (seconds):** `604800` (7 days)

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
  ## Troubleshooting

### "MongoDB memory could not create its indexes"

The connection cannot create indexes on the target database. Grant the user
index-creation rights, or create the indexes out of band. The node refuses to
start rather than run without them, because a memory without its indexes
degrades into full collection scans as it grows.

### Long-term items are being read as checkpoints, or vice versa

**Long-term Collection** is pointing at one of the checkpoint collections.
Give each a distinct name.

### Conversations vanish after a while

The TTL index is expiring them. Clear **TTL (seconds)** to keep them, or raise
it.

### Two environments are overwriting each other

They share a database and the default collection names. Set **Database** or
the three collection fields per environment.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
  ## Related Nodes

- **Agent V2** — the node this attaches to.
- **Postgres Memory**, **Redis Memory** — the other durable, worker-shared
  options.
- **Memory** — in-process, for development.

<!-- /SECTION: related -->

---
  node_id: "postgres-memory"
  title: "Postgres Memory"
  description: "Durable conversation memory, execution checkpoints and long-term recall in your own Postgres database, reachable from every worker."
  category: "generative-ai"
  subcategory: "memory"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-09-02"
  author: "Fusion Team"
  tags:
    - memory
    - postgres
    - agent
    - conversation
    - checkpoints
  related_nodes:
    - agent-v2
    - memory
    - redis-memory
---

<!-- SECTION: overview -->
  # Postgres Memory

  > **Category:** Generative AI &amp; LLMs&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Memory Node

  Conversation memory in your own Postgres database. Connect it to an Agent V2
  node's **Memory** handle.

  Durable **and** reachable from every worker, so a conversation survives a
  restart and survives the workflow moving between replicas.

### What it provides

| Capability | Provided | What it means |
|---|---|---|
| Conversation | Yes | The transcript the chat displays. |
| Checkpoints | Yes | An interrupted run resumes where it stopped. |
| Long-term recall | Yes | Values kept across separate conversations. |

### Use Cases

- A production assistant whose history must outlive deploys
- A multi-replica deployment where any worker may pick up the next turn
- A team that already runs Postgres and wants one less service to operate

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
  ## Configuration

| Field | Required | Default | Description |
|---|---|---|---|
| **Connection String** | Yes | — | `postgresql://user:password@host:5432/database`. Needs CREATE TABLE permission the first time this workflow runs. |
| **Schema** | No | `public` | Postgres schema for the checkpoint and store tables. |
| **Create Tables Automatically** | No | `true` | Turn off if your DBA runs migrations out of band. The tables must already exist. |
| **Default TTL (minutes)** | No | — | Leave empty to keep long-term items forever. |
| **Refresh TTL On Read** | No | `true` | Reading an item restarts its expiry clock. |
| **Sweep Interval (minutes)** | No | `60` | How often expired items are deleted. |

### The first run needs CREATE TABLE

The node creates its tables and indexes the first time it starts. A
read/write-only credential works on every subsequent run and fails that one,
which is a confusing way to find out — so grant DDL for the first run, or turn
**Create Tables Automatically** off and have your DBA create the tables.

### Retention

TTL applies to **long-term items**, not to the transcript. Leave it empty and
nothing expires. The sweep is Postgres-side, on the interval you set here.

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
  mem[(Postgres Memory)] -- memory --> agent
```

Configuration:

- **Connection String:** `postgresql://fusion:••••@postgres:5432/fusion`
- **Schema:** `agent`
- **Default TTL (minutes):** `43200` (30 days)

The builder shows **Durable memory** on the agent — no warning, because this
memory both survives a restart and is reachable from any worker.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
  ## Troubleshooting

### "Postgres memory could not create its tables in schema ..."

The connection lacks CREATE TABLE permission. Grant it for the first run, or
set **Create Tables Automatically** off and create the tables yourself.

### The agent works but forgets across conversations

Long-term items have expired. Check **Default TTL (minutes)** — leave it empty
to keep them forever.

### Two workflows are sharing a conversation

Each memory is scoped to the workflow, the memory node, the agent and the
participant, so this should not happen from duplication. It can happen if the
same exported workflow was imported twice, since import preserves node ids.
Give one of them a fresh memory node.

### Tables appear in the wrong schema

**Schema** defaults to `public`. Set it explicitly when sharing one database
across environments.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
  ## Related Nodes

- **Agent V2** — the node this attaches to.
- **Redis Memory**, **MongoDB Memory** — the other durable, worker-shared
  options.
- **Memory** — in-process, for development.

<!-- /SECTION: related -->

---
  node_id: "redis-memory"
  title: "Redis Memory"
  description: "Durable conversation memory, execution checkpoints and long-term recall in your own Redis, with native key expiry."
  category: "generative-ai"
  subcategory: "memory"
  version: "1.0.0"
  language: "en"
  last_updated: "2026-09-02"
  author: "Fusion Team"
  tags:
    - memory
    - redis
    - agent
    - conversation
    - checkpoints
  related_nodes:
    - agent-v2
    - memory
    - postgres-memory
---

<!-- SECTION: overview -->
  # Redis Memory

  > **Category:** Generative AI &amp; LLMs&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Memory Node

  Conversation memory in your own Redis. Connect it to an Agent V2 node's
  **Memory** handle.

  Durable and reachable from every worker. Redis is the one backend where
  expiry is native, so retention costs nothing to run — there is no sweep to
  schedule.

### What it provides

| Capability | Provided | What it means |
|---|---|---|
| Conversation | Yes | The transcript the chat displays. |
| Checkpoints | Yes | An interrupted run resumes where it stopped. |
| Long-term recall | Yes | Values kept across separate conversations. |

### Use Cases

- A high-traffic assistant where memory reads must be fast
- A deployment that already runs Redis for queues or caching
- Conversations that should expire on their own after a set period

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
  ## Configuration

| Field | Required | Default | Description |
|---|---|---|---|
| **Redis URL** | Yes | — | `redis://user:password@host:6379/0`. |
| **Cluster Node URLs** | No | — | Leave empty for a single Redis. Any entry here switches to cluster mode. |
| **Default TTL (minutes)** | No | — | Leave empty to keep checkpoints until you delete them. |
| **Refresh TTL On Read** | No | `true` | Reading a checkpoint restarts its expiry clock. |

### Cluster mode

**Any entry in Cluster Node URLs switches this node to cluster mode, and the
single Redis URL is then ignored.** Fill in one or the other, not both — a URL
left behind in the single field is silently unused, which is a confusing way to
connect to the wrong Redis.

### Retention

Redis expires keys itself, so **Default TTL (minutes)** is enforced by the
server rather than by a background sweep. With **Refresh TTL On Read** on, an
active conversation keeps renewing itself and only an idle one expires.

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

```
Chat Trigger ──▶ Agent V2 ──▶ Send Reply
                   ▲  ▲
        OpenAI LLM ┘  └ Redis Memory
```

Configuration:

- **Redis URL:** `redis://default:••••@redis:6379`
- **Default TTL (minutes):** `10080` (7 days)
- **Refresh TTL On Read:** on

An active conversation renews itself; one nobody returns to expires after a
week.

<!-- /SECTION: workflow-example -->

<!-- SECTION: troubleshooting -->
  ## Troubleshooting

### Conversations vanish after a while

That is the TTL doing its job. Clear **Default TTL (minutes)** to keep them
indefinitely, or raise it.

### It connects to the wrong Redis

Check **Cluster Node URLs**. Any entry there puts the node in cluster mode and
the single **Redis URL** is ignored.

### The connection is refused or reports an authentication error

Redis with a password set rejects unauthenticated clients. Put the credentials
in the URL: `redis://default:password@host:6379`.

### Long-term recall fails but chat works

The long-term store needs Redis's search module, which is built into Redis 8
and shipped separately for older versions. On an older Redis, use Redis Stack
or switch to Postgres or MongoDB memory.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
  ## Related Nodes

- **Agent V2** — the node this attaches to.
- **Postgres Memory**, **MongoDB Memory** — the other durable, worker-shared
  options.
- **Memory** — in-process, for development.

<!-- /SECTION: related -->

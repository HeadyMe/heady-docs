# Colab 3-Runtime Control Plane & Latent-Space Operations

> Heady Systems LLC — Design, lifecycle, and operational model for the 3-node Colab GPU cluster.

---

## Overview

Heady's compute layer runs on Google Colab Pro+ as a **3-runtime distributed GPU cluster**. Each runtime has a specific role in the Latent OS lifecycle. They are connected via Tailscale mesh VPN and coordinated through Redis (Upstash) pub/sub.

---

## Runtime Roles

| Runtime | Codename | GPU | RAM | Primary Role |
|---|---|---|---|---|
| **Node 0** | Overmind | T4 (16 GB VRAM) | 51 GB | Cognitive orchestration, directive routing, Socratic Loop |
| **Node 1** | Forge | T4 (16 GB VRAM) | 51 GB | Code generation, Battle Arena execution, AST projection |
| **Node 2** | Edge | A100 (40 GB VRAM) | 80 GB | Embedding, vector memory, heavy inference, fine-tuning |

An optional **Node 3** (T4 or A100) can be spun up for burst compute and overflow.

---

## Control Plane Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                    heady-manager (Cloud Run)                   │
│                    Control Plane Coordinator                   │
│  ┌─────────┐  ┌──────────┐  ┌───────────┐  ┌──────────────┐ │
│  │ /health │  │ /dispatch │  │ /status   │  │ /scale       │ │
│  │ endpoint│  │ task queue│  │ node state│  │ burst toggle │ │
│  └────┬────┘  └────┬─────┘  └─────┬─────┘  └──────┬───────┘ │
└───────┼────────────┼──────────────┼────────────────┼─────────┘
        │            │              │                │
        │     ┌──────┴──────────────┴────────────────┴──┐
        │     │           Redis (Upstash)                │
        │     │   Pub/Sub channels + task queues         │
        │     └──────┬──────────────┬───────────────┬───┘
        │            │              │               │
   ┌────┴─────┐ ┌───┴─────┐  ┌────┴─────┐  ┌─────┴─────┐
   │ Node 0   │ │ Node 1  │  │ Node 2   │  │ Node 3    │
   │ Overmind │ │ Forge   │  │ Edge     │  │ (burst)   │
   │ T4/51GB  │ │ T4/51GB │  │ A100/80GB│  │ on-demand │
   └──────────┘ └─────────┘  └──────────┘  └───────────┘
        │            │              │               │
        └────────────┴──────────────┴───────────────┘
                   Tailscale Mesh VPN
                (encrypted peer-to-peer)
```

---

## Latent-Space Operations

The Colab cluster is the execution engine for the **Continuous Latent Architecture (CLA)**. Code and knowledge live as vectors in pgvector/Neon; the Colab nodes transform and project them.

### Core Operations

| Operation | Node | Description |
|---|---|---|
| **Embed** | Edge (Node 2) | Convert text/code to 384D embeddings via `all-MiniLM-L6-v2`, then PCA-project to (x, y, z) |
| **Store** | Edge (Node 2) | Write vectors to pgvector with octant zone tags and Fibonacci shard assignment |
| **Retrieve** | Edge (Node 2) | Zone-first semantic search: filter by octant, then cosine similarity within zone |
| **Reason** | Overmind (Node 0) | Socratic Loop: multi-model reasoning with ternary logic (+1, 0, -1) and epistemic hold |
| **Generate** | Forge (Node 1) | Produce JSON AST from vector context; Battle Arena runs competing model outputs |
| **Project** | Forge (Node 1) | Convert JSON AST → source code; push to GitHub template repos via `liquid-deploy.js` |
| **Validate** | Overmind (Node 0) | 3 Unbreakable Laws check: no secrets, 100% completion, vectorize all interactions |

### Vector Memory Model

```
Embedding Pipeline:
  raw input → all-MiniLM-L6-v2 (384D) → PCA-lite → (x, y, z)

Storage:
  pgvector/Neon → 8-octant spatial index → Fibonacci sharding (5 shards)

Retrieval:
  query → embed → octant filter → cosine similarity → top-k results
  Latency target: <50ms for zone-first retrieval
```

### Octant Zones

The 3D vector space is divided into 8 octants based on sign of (x, y, z):

| Octant | Signs | Semantic Region |
|---|---|---|
| 0 | (+, +, +) | Core platform logic |
| 1 | (+, +, −) | User-facing features |
| 2 | (+, −, +) | Infrastructure & ops |
| 3 | (+, −, −) | Security & compliance |
| 4 | (−, +, +) | Creative & design |
| 5 | (−, +, −) | Research & analysis |
| 6 | (−, −, +) | Trading & finance |
| 7 | (−, −, −) | Experimental / staging |

---

## Runtime Lifecycle

### Startup Sequence

1. **Boot**: Colab notebook mounts Google Drive, installs deps (`pip install tailscale-client redis pgvector`)
2. **Network**: Join Tailscale mesh, register node with heady-manager health endpoint
3. **Announce**: Publish `node:online` to Redis channel `heady:control` with GPU type and available VRAM
4. **Ready**: Begin polling Redis task queue for assigned work

### Health Monitoring

Each node publishes a heartbeat every 30 seconds to `heady:heartbeat:<node_id>`:

```json
{
  "node": "overmind",
  "gpu": "T4",
  "vram_used_gb": 8.2,
  "vram_total_gb": 16,
  "ram_used_gb": 22,
  "ram_total_gb": 51,
  "active_tasks": 3,
  "uptime_s": 14400,
  "ts": "2026-03-10T14:30:00Z"
}
```

`heady-manager` monitors heartbeats. If a node misses 3 consecutive beats (90 seconds), it is marked `degraded` and tasks are rerouted.

### Shutdown / Preemption

Colab runtimes can be preempted by Google. The graceful shutdown sequence:

1. Node receives `SIGTERM` or detects idle timeout
2. Publishes `node:draining` to Redis
3. Completes in-flight tasks (max 60s grace period)
4. Publishes `node:offline`
5. heady-manager redistributes queued tasks to remaining nodes

---

## Task Dispatch

Tasks are dispatched from `heady-manager` via Redis queues, with routing based on task type and node capability.

### Queue Structure

| Queue | Target Node | Task Types |
|---|---|---|
| `heady:tasks:overmind` | Node 0 | reasoning, validation, directive routing |
| `heady:tasks:forge` | Node 1 | code generation, AST projection, Battle Arena |
| `heady:tasks:edge` | Node 2 | embedding, vector store/retrieve, fine-tuning |
| `heady:tasks:any` | Any available | overflow, burst tasks |

### Task Envelope

```json
{
  "task_id": "task_abc123",
  "type": "embed",
  "priority": 1,
  "payload": { "text": "...", "metadata": {} },
  "created_at": "2026-03-10T14:30:00Z",
  "timeout_ms": 30000,
  "retry_count": 0,
  "max_retries": 2
}
```

### Retry & Circuit Breaker

- Failed tasks retry up to `max_retries` with exponential backoff + jitter
- If a node's failure rate exceeds 50% in a 5-minute window, the circuit breaker opens and tasks route to other nodes
- Circuit breaker resets after 60 seconds of successful health checks

---

## Inter-Node Communication

| Channel | Protocol | Purpose |
|---|---|---|
| Tailscale mesh | WireGuard (encrypted) | Direct peer-to-peer data transfer between nodes |
| Redis pub/sub | TLS to Upstash | Control plane messages, heartbeats, task dispatch |
| pgvector/Neon | TLS PostgreSQL | Shared vector memory (all nodes read/write) |

### Data Flow Example: User Query

```
1. User sends query via MCP SSE (heady.headyme.com)
2. heady-manager receives, publishes to heady:tasks:overmind
3. Overmind (Node 0) picks up, runs Socratic Loop reasoning
4. If code generation needed → publishes sub-task to heady:tasks:forge
5. Forge (Node 1) generates JSON AST, writes to pgvector
6. If embedding needed → publishes to heady:tasks:edge
7. Edge (Node 2) embeds result, stores in 3D vector space
8. Overmind aggregates results, returns via Redis → heady-manager → SSE
```

---

## Operational Runbook

### Scaling Up (Burst Node)

1. Open new Colab Pro+ notebook
2. Run standard boot cell (mounts drive, joins Tailscale, connects Redis)
3. Node auto-registers and begins accepting `heady:tasks:any` overflow

### Monitoring

- **Redis**: `SUBSCRIBE heady:heartbeat:*` to watch all node health
- **heady-manager**: `GET /status` returns aggregated cluster state
- **Sentry**: Runtime errors are captured via `SENTRY_DSN` with node tags

### Cost Model

| Resource | Cost | Notes |
|---|---|---|
| Colab Pro+ | $49.99/mo | 3–4 concurrent runtimes, priority GPU access |
| Upstash Redis | ~$10/mo | Serverless, pay-per-command |
| Neon Postgres (Scale) | ~$25/mo | Autoscaling, pgvector enabled |
| Tailscale | Free (personal) | Up to 100 devices |
| **Total** | **~$85/mo** | Full GPU cluster with coordination |

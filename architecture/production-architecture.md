# Production Architecture Navigation

> Heady Systems LLC — How the production pieces connect, and where to find them.

---

## System Map

This document is the operator's guide to Heady's production architecture. It maps every major component to its location, its upstream and downstream dependencies, and the repo where its code lives.

```
                          ┌─────────────────────┐
                          │     User / IDE       │
                          │  (Browser, Cursor,   │
                          │   VS Code, CLI)      │
                          └──────────┬──────────┘
                                     │
                          ┌──────────┴──────────┐
                          │   Cloudflare Edge    │
                          │   heady-edge-node    │
                          │   (12+ domains)      │
                          │   ┌───────────────┐  │
                          │   │ CORS enforce  │  │
                          │   │ Rate limit    │  │
                          │   │ Route + sign  │  │
                          │   └───────────────┘  │
                          └──────────┬──────────┘
                                     │
               ┌─────────────────────┼─────────────────────┐
               │                     │                     │
    ┌──────────┴──────┐   ┌─────────┴─────────┐   ┌──────┴──────────┐
    │ Static Sites    │   │  heady-manager     │   │  HeadyWeb       │
    │ (CF Pages)      │   │  (Cloud Run)       │   │  (Cloud Run /   │
    │ systems/connect │   │  ┌──────────────┐  │   │   Vercel)       │
    │ /buddy          │   │  │ REST API     │  │   └─────────────────┘
    └─────────────────┘   │  │ MCP SSE      │  │
                          │  │ OAuth flows  │  │
                          │  │ Webhook rx   │  │
                          │  │ Task dispatch│  │
                          │  └──────┬───────┘  │
                          └─────────┼──────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
             ┌──────┴─────┐ ┌──────┴─────┐ ┌──────┴─────┐
             │  Neon PG   │ │  Upstash   │ │  Colab     │
             │  (pgvector)│ │  Redis     │ │  Cluster   │
             │  vectors,  │ │  queues,   │ │  3 GPUs    │
             │  users,    │ │  pub/sub,  │ │  see: colab│
             │  keys      │ │  heartbeat │ │  -control- │
             └────────────┘ └────────────┘ │  plane.md  │
                                           └────────────┘
```

---

## Component Reference

### Edge Layer

| Component | Technology | Repo / Location | Purpose |
|---|---|---|---|
| **heady-edge-node** | Cloudflare Worker | `heady-production` → `workers/` | Origin routing, CORS, rate limiting, request signing |
| **DNS** | Cloudflare DNS | Cloudflare dashboard | All 12 domains managed centrally |
| **SSL** | Cloudflare Universal SSL | Automatic | TLS termination at edge |
| **DDoS** | Cloudflare WAF | Cloudflare dashboard | Layer 3/4/7 protection |

### Application Layer

| Component | Technology | Repo / Location | Purpose |
|---|---|---|---|
| **heady-manager** | Node.js / Express | `heady-production` → `src/` | Unified backend: API, MCP SSE, auth, task dispatch |
| **liquid-deploy.js** | Node.js | `heady-production` → `src/` | Synaptic Forge: AST → GitHub projection |
| **structured-logger.js** | Node.js | `heady-production` → `src/` | JSON telemetry with circuit breakers |
| **HeadyWeb** | React (Module Federation) | `HeadyWeb` repo | Web portal, OAuth login, dashboard |
| **HeadyBuddy** | Chrome Extension | `HeadyBuddy` repo | Browser companion |
| **HeadyAI-IDE** | VS Code Extension | `HeadyAI-IDE` repo | IDE integration via MCP |
| **admin-ui** | React | `admin-ui` repo | Admin dashboard |

### Data Layer

| Component | Technology | Purpose |
|---|---|---|
| **Neon Postgres** | PG 16 + pgvector | Vector memory, user accounts, API keys, audit log |
| **Upstash Redis** | Serverless Redis | Task queues, pub/sub, session cache, heartbeats |
| **Pinecone** | Managed vector DB | Distributed vector search (secondary index) |

### Compute Layer

| Component | Technology | Purpose |
|---|---|---|
| **Colab Cluster** | 3× Google Colab Pro+ | GPU inference, embedding, code gen — see [Colab Control Plane](./colab-control-plane.md) |
| **Tailscale Mesh** | WireGuard VPN | Encrypted inter-node networking |

---

## Request Lifecycle (End-to-End)

A typical MCP tool invocation from an IDE:

```
1. IDE sends request to heady.headyme.com/mcp/sse
   Headers: Authorization: Bearer <api_key>

2. Cloudflare Edge Worker (heady-edge-node):
   ├─ Validates origin against ALLOWED_ORIGINS
   ├─ Rate-limit check (token bucket, per API key tier)
   ├─ Signs request for origin authentication
   └─ Forwards to Cloud Run (heady-manager)

3. heady-manager:
   ├─ Validates API key (DB lookup in Neon)
   ├─ Checks tier scopes and permissions
   ├─ Resolves MCP tool from request
   ├─ If local tool: executes inline
   └─ If GPU tool: dispatches to Colab via Redis

4. Colab Node (if dispatched):
   ├─ Picks task from Redis queue
   ├─ Executes (embed / generate / reason)
   ├─ Writes results to pgvector and/or Redis
   └─ Publishes completion event

5. heady-manager:
   ├─ Receives completion from Redis
   ├─ Formats SSE response
   └─ Streams back to IDE via SSE

6. Edge Worker:
   └─ Passes response through (no modification)
```

---

## Repo Map

| Repository | What Lives There | Link |
|---|---|---|
| **heady-production** | Monorepo: heady-manager, edge workers, liquid-deploy, MCP tools, configs | Primary codebase |
| **HeadyWeb** | React web portal (headyapp.com) | Consumer of heady-manager API |
| **HeadyBuddy** | Chrome extension | Consumer of heady-manager API |
| **HeadyAI-IDE** | VS Code / IDE extension | MCP SSE client |
| **admin-ui** | Admin dashboard | Consumer of heady-manager API |
| **heady-docs** | Documentation hub (this repo) | Docs, patents, architecture |
| **latent-core-dev** | Synaptic Dev (pgvector ↔ Antigravity) | Experimental |
| **template-mcp-server** | MCP server boilerplate | Template |
| **template-swarm-bee** | Swarm agent boilerplate | Template |
| **template-heady-ui** | React micro-frontend boilerplate | Template |
| **heady-rebuild-*** (9) | Battle Arena competitive rebuilds | Evaluation |

---

## Key Configuration Files

| File / Variable | Location | Purpose |
|---|---|---|
| `.env` | heady-production root (not committed) | All secrets and config — see [API Keys Reference](../api/api-keys-reference.md) |
| `ALLOWED_ORIGINS` | .env + edge worker | CORS allowlist |
| `DATABASE_URL` | .env | Neon Postgres connection string |
| `UPSTASH_REDIS_REST_URL` | .env | Redis endpoint |
| `SENTRY_DSN` | .env | Error tracking |
| `CLOUDFLARE_API_TOKEN` | .env | Edge worker deployment |
| `GITHUB_TOKEN` | .env | liquid-deploy.js GitHub pushes |

---

## Operational Quick Reference

| Task | How |
|---|---|
| **Check cluster health** | `GET heady-manager/status` or `SUBSCRIBE heady:heartbeat:*` in Redis |
| **Deploy edge worker** | `wrangler deploy` from heady-production/workers/ |
| **Deploy heady-manager** | `gcloud run deploy` from heady-production root |
| **Rotate API key** | headyme.com → Settings → API Keys → Regenerate |
| **View logs** | Sentry dashboard (errors) or Cloud Run logs (all) |
| **Scale Colab** | Open new notebook, run boot cell — auto-registers |
| **Force drain node** | Publish `{"action":"drain"}` to `heady:control:<node_id>` |

---

## Cross-References

- [Site & Domain Registry](./site-domain-registry.md) — Domain routing and CORS config
- [Auth Contract](./auth-contract.md) — Authentication methods and token lifecycle
- [Colab Control Plane](./colab-control-plane.md) — GPU cluster operations
- [Service Catalog](../sources/04-heady-service-catalog-and-capabilities.md) — 30+ MCP tools
- [Architecture Patterns](../sources/05-heady-architecture-and-patterns.md) — Six-layer stack and design patterns
- [API Keys Reference](../api/api-keys-reference.md) — Service credentials inventory

# Heady™ Documentation Hub

> **Single Source of Truth** for all Heady project documentation

[![Patents](https://img.shields.io/badge/patents-51%2B-blue)](./03_Patents/)
[![Repos](https://img.shields.io/badge/repos-18-green)](#github-repository-ecosystem)
[![MCP Tools](https://img.shields.io/badge/MCP%20tools-30%2B-orange)](./02_Services/service-catalog.md)

---

## Quick Start

1. **New to Heady?** Start with the [Executive Overview](./00_Overview/executive-overview.md)
2. **Understand the stack** — Read the [Architecture & Patterns](./02_Services/architecture-and-patterns.md)
3. **Browse services** — See the [Service Catalog](./02_Services/service-catalog.md) (30+ MCP tools)
4. **Clone the monorepo** — [Heady-pre-production](https://github.com/HeadyMe/Heady-pre-production-9f2f0642)

---

## Vault Structure

```
heady-docs/
├── 00_Overview/        — Platform vision, comprehensive source (v3.1)
├── 01_Architecture/    — Production system map, domains, auth, Colab cluster
├── 02_Services/        — 30+ MCP tools catalog and design patterns
├── 03_Patents/         — 51+ USPTO provisional applications (full-text Batch 4)
├── 04_API_Reference/   — Service inventory and environment variable reference
├── 05_Strategic/       — Valuation, market research, investor targets
├── 06_Trading/         — APEX autonomous trading intelligence
├── 07_IP_Portfolio/    — IP portfolio analysis and strategic valuation
├── Assets/             — Shared media and design assets
├── site/               — Pre-built static HTML/CSS documentation hub
└── sources/            — Legacy sources (HEADY_KNOWLEDGE_VAULT.md preserved)
```

---

## Documentation Index

| Section | Description |
| --- | --- |
| [00 Overview](./00_Overview/_README.md) | Platform vision, philosophy, and comprehensive source (v3.1) |
| [Executive Overview](./00_Overview/executive-overview.md) | Platform vision and market position |
| [Comprehensive Source](./00_Overview/comprehensive-source.md) | Full platform deep-dive (v3.1) |
| [01 Architecture](./01_Architecture/_README.md) | Production infrastructure deep-dives |
| [Production Architecture](./01_Architecture/production-architecture.md) | End-to-end system map and operator navigation guide |
| [Site & Domain Registry](./01_Architecture/site-domain-registry.md) | Canonical registry of all 12 domains, routing, and CORS |
| [Auth Contract](./01_Architecture/auth-contract.md) | Cross-site auth model — API keys, OAuth PKCE, HMAC webhooks |
| [Colab Control Plane](./01_Architecture/colab-control-plane.md) | 3-runtime GPU cluster, latent-space ops, task dispatch |
| [02 Services](./02_Services/_README.md) | MCP tool catalog and service design patterns |
| [Service Catalog](./02_Services/service-catalog.md) | 30+ MCP tools across 7 domains |
| [Architecture & Patterns](./02_Services/architecture-and-patterns.md) | Six-layer stack and design patterns |
| [03 Patents](./03_Patents/_README.md) | 51+ USPTO provisional applications |
| [Patent Index](./03_Patents/patent-index.md) | Full portfolio index with filing details |
| [04 API Reference](./04_API_Reference/_README.md) | Service inventory and environment setup |
| [API Keys Reference](./04_API_Reference/api-keys-reference.md) | 14 external services with env var names |
| [05 Strategic](./05_Strategic/_README.md) | Valuation models and investor intelligence |
| [Value Assessment Q1 2026](./05_Strategic/value-assessment-2026-q1.md) | Q1 2026 valuation and growth projections |
| [Knowledge Vault](./05_Strategic/knowledge-vault/README.md) | Notion-synced project data package |
| [06 Trading](./06_Trading/_README.md) | Autonomous trading intelligence |
| [APEX Trading Intelligence](./06_Trading/apex-trading-intelligence.md) | APEX autonomous trading architecture |
| [07 IP Portfolio](./07_IP_Portfolio/_README.md) | IP portfolio analysis and valuation |
| [IP Portfolio & Valuation](./07_IP_Portfolio/ip-portfolio-and-valuation.md) | 51+ patents, valuation, and strategy |

---

## Architecture at a Glance

```
Layer 1  Edge        — Cloudflare CDN, DDoS, Edge Workers, SSL (12 domains)
Layer 2  Gateway     — Auth (HEADY_API_KEY), Rate Limiting, mTLS
Layer 3  Orchestrate — Swarm Conductor, Bee Factory, Task Routing
Layer 4  Intelligence — Multi-Model (GPT-4o, Claude, Gemini, Groq), Ternary Logic
Layer 5  Memory      — 3D Vectors (Pinecone), pgvector (Neon), Embeddings
Layer 6  Persistence — Neon Postgres, Audit Logs, Config Store
```

---

## Authentication Model

- **API Gateway**: All requests authenticate via `HEADY_API_KEY` at Layer 2
- **Admin**: Elevated operations require `ADMIN_TOKEN`
- **Edge Security**: Cloudflare handles SSL, DDoS, and rate limiting at Layer 1
- **Service-to-Service**: Scoped tokens with least-privilege access
- **Secrets**: Environment variables only — never hardcoded. 1Password SA for managed rotation.

See [API & Services Reference](./04_API_Reference/api-keys-reference.md) for the full service inventory.

---

## Deployment Stack

| Component | Service | Role |
| --- | --- | --- |
| CDN / Edge | Cloudflare Workers | 12 domains, dynamic delivery, DNS |
| API / Backend | Google Cloud Run | heady-manager, serverless MCP bridge |
| Database | Neon Postgres (Scale) | PG 16, pgvector, 5 tables |
| Vector Store | Pinecone | Distributed vector DB for embeddings |
| GPU Compute | 3-Node Colab Pro+ | Overmind, Forge, Edge nodes |
| Error Tracking | Sentry | Project: heady-manager |
| Secrets | 1Password SA | Service account token management |
| Payments | Stripe (Live) | Subscription billing |

---

## Connected Sources

### NotebookLM

All documents in `00_Overview/` are optimized for NotebookLM ingestion:
- Upload to create cinematic explainer videos
- Deep-dive audio overviews
- Interactive Q&A sessions

### GitHub Repository Ecosystem

| Repository | Purpose | Visibility |
| --- | --- | --- |
| [Heady-pre-production](https://github.com/HeadyMe/Heady-pre-production-9f2f0642) | Official monorepo — single source of truth | Public |
| [heady-docs](https://github.com/HeadyMe/heady-docs) | Documentation hub (this repo) | Public |
| latent-core-dev | Synaptic Dev — pgvector ↔ Antigravity | Private |
| template-mcp-server | Template — MCP protocol server | Private |
| template-swarm-bee | Template — Swarm Agent lifecycle | Private |
| template-heady-ui | Template — React Module Federation | Private |
| HeadyBuddy | Chrome Extension | Private |
| HeadyWeb | Web portal | Private |
| HeadyAI-IDE | IDE Extension | Private |
| admin-ui | Admin dashboard | Private |

**Battle Arena** (9 repos): Competitive rebuilds — Groq, Claude, Gemini, GPT-5.4, Codex, Perplexity, HeadyCoder, HuggingFace, Jules

---

## Deploy as Static Site

```bash
# GitHub Pages: Settings → Pages → Source: main → /site
# Or use Cloudflare Pages / Vercel pointing to /site
```

The `/site` directory contains a pre-built static HTML/CSS documentation hub. No build step required.

---

*© 2026 Heady Systems LLC. All rights reserved.*

# Heady™ Documentation Hub

> **Single Source of Truth** for all Heady project documentation

[![Patents](https://img.shields.io/badge/patents-51%2B-blue)](./patents/)
[![Repos](https://img.shields.io/badge/repos-18-green)](#github-repository-ecosystem)
[![MCP Tools](https://img.shields.io/badge/MCP%20tools-30%2B-orange)](./sources/04-heady-service-catalog-and-capabilities.md)

---

## Quick Start

1. **New to Heady?** Start with the [Executive Overview](./sources/01-heady-executive-overview.md)
2. **Understand the stack** — Read the [Architecture & Patterns](./sources/05-heady-architecture-and-patterns.md)
3. **Browse services** — See the [Service Catalog](./sources/04-heady-service-catalog-and-capabilities.md) (30+ MCP tools)
4. **Clone the monorepo** — [Heady-pre-production](https://github.com/HeadyMe/Heady-pre-production-9f2f0642)

---

## Documentation Index

| Section | Description |
| --- | --- |
| [Comprehensive Source](./sources/00-comprehensive-source.md) | Full platform deep-dive (v3.1) |
| [Executive Overview](./sources/01-heady-executive-overview.md) | Platform vision and market position |
| [Trading Intelligence](./sources/02-heady-apex-trading-intelligence.md) | APEX autonomous trading architecture |
| [IP Portfolio](./sources/03-heady-ip-portfolio-and-valuation.md) | 51+ patents, valuation, and strategy |
| [Service Catalog](./sources/04-heady-service-catalog-and-capabilities.md) | 30+ MCP tools across 7 domains |
| [Architecture](./sources/05-heady-architecture-and-patterns.md) | Six-layer stack and design patterns |
| [Patent Portfolio](./patents/README.md) | Full patent index with filing details |
| [API & Services Reference](./api/api-keys-reference.md) | Service inventory and environment setup |
| [Strategic Value Assessment](./strategic/value-assessment-2026-q1.md) | Q1 2026 valuation and growth projections |

### Architecture Deep-Dives

| Section | Description |
| --- | --- |
| [Production Architecture](./architecture/production-architecture.md) | End-to-end system map and operator navigation guide |
| [Site & Domain Registry](./architecture/site-domain-registry.md) | Canonical registry of all 12 domains, routing, and CORS |
| [Auth Contract](./architecture/auth-contract.md) | Cross-site auth model — API keys, OAuth PKCE, HMAC webhooks |
| [Colab Control Plane](./architecture/colab-control-plane.md) | 3-runtime GPU cluster, latent-space ops, task dispatch |

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

See [API & Services Reference](./api/api-keys-reference.md) for the full service inventory.

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

All documents in `sources/` are optimized for NotebookLM ingestion:
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

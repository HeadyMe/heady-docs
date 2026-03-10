# Heady™ Documentation Hub

> **Single Source of Truth** for all Heady project documentation

[![Patents](https://img.shields.io/badge/patents-51%2B-blue)](./patents/)
[![Repos](https://img.shields.io/badge/repos-18-green)](./README.md#-github-repository-ecosystem)

---

## 🚀 Getting Started

New to Heady? Start here:

| Guide | Description |
| --- | --- |
| [**Quickstart**](./docs/getting-started/quickstart.md) | 5-step guide to connecting and using Heady |
| [**Authentication**](./docs/reference/authentication.md) | API keys, OAuth, scopes, and cross-domain auth |
| [**Glossary**](./docs/reference/glossary.md) | Definitions for Heady-specific terminology |

---

## 📚 Documentation Index

### Platform & Architecture

| Document | Description |
| --- | --- |
| [Comprehensive Source](./sources/00-comprehensive-source.md) | Full platform deep-dive (v3.1) |
| [Executive Overview](./sources/01-heady-executive-overview.md) | Platform vision and market position |
| [Architecture & Patterns](./sources/05-heady-architecture-and-patterns.md) | Six-layer stack and 24+ design patterns |
| [Service Catalog](./sources/04-heady-service-catalog-and-capabilities.md) | 30+ MCP tools across 7 domains |

### Trading & IP

| Document | Description |
| --- | --- |
| [Trading Intelligence](./sources/02-heady-apex-trading-intelligence.md) | APEX autonomous trading architecture |
| [IP Portfolio](./sources/03-heady-ip-portfolio-and-valuation.md) | 51+ patents, valuation, and strategy |
| [Patent Portfolio](./patents/README.md) | Full patent index with filing details |
| [Strategic Valuation](./strategic/value-assessment-2026-q1.md) | Q1 2026 financial assessment |

### Reference

| Document | Description |
| --- | --- |
| [API Keys Reference](./api/api-keys-reference.md) | Environment variable inventory by service |
| [Authentication](./docs/reference/authentication.md) | Auth methods, scopes, token lifecycle |
| [Glossary](./docs/reference/glossary.md) | Platform terminology and concepts |

---

## 🌐 Static Site

The `site/` directory contains a polished landing page. To deploy:

```bash
# GitHub Pages: Settings → Pages → Source: main → /site
# Or use Cloudflare Pages / Vercel pointing to /site
```

---

## 🔗 Connected Sources

### NotebookLM

All documents in `sources/` are optimized for NotebookLM ingestion:

- Upload to create cinematic explainer videos
- Deep-dive audio overviews
- Interactive Q&A sessions

### GitHub Ecosystem (18 Repos)

- **Monorepo:** [Heady-pre-production-9f2f0642](https://github.com/HeadyMe/Heady-pre-production-9f2f0642)
- **Templates:** mcp-server, swarm-bee, heady-ui
- **Battle Arena:** 9 competitive rebuild repos (Groq, Claude, Gemini, GPT-5.4, Codex, Perplexity, HeadyCoder, HuggingFace, Jules)
- **Products:** HeadyBuddy, HeadyWeb, HeadyAI-IDE, admin-ui

---

*© 2026 Heady Systems LLC. All rights reserved.*

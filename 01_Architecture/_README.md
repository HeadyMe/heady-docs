---
id: 01_architecture
tags: [docs, heady, architecture]
---
# 01 — Architecture

Deep-dive technical documentation for Heady's production infrastructure. Covers the end-to-end system topology, all managed domains, the cross-site authentication model, and the 3-node Colab GPU cluster that powers latent-space operations.

## Contents

- [Production Architecture](./production-architecture.md) — Operator navigation guide: full system map with component-to-repo mapping, upstream/downstream dependencies, and the 6-layer stack in context
- [Site & Domain Registry](./site-domain-registry.md) — Canonical registry of all 12 Heady domains, Cloudflare routing rules, CORS policy, and auth surface per domain
- [Auth Contract](./auth-contract.md) — Cross-site authentication model: API key (Bearer), OAuth 2.0 PKCE, and HMAC webhook signature — token lifecycle, storage rules, and implementation scaffolding
- [Colab Control Plane](./colab-control-plane.md) — 3-runtime GPU cluster design (Overmind / Forge / Edge nodes), Tailscale mesh VPN, Redis pub/sub coordination, and latent-space ops lifecycle

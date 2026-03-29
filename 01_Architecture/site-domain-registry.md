# Canonical Site & Domain Registry

> Heady Systems LLC — Single source of truth for all domains, surfaces, and their routing.

---

## Domain Inventory

Every Heady domain is fronted by Cloudflare. DNS, SSL termination, and DDoS protection are handled at the edge before traffic reaches any origin.

| Domain | Purpose | Origin | Auth Surface |
|---|---|---|---|
| **headyme.com** | Primary developer portal & login | GCP Cloud Run (`heady-manager`) | OAuth + API key |
| **headysystems.com** | Enterprise / corporate | Cloudflare Pages (static) | None (public) |
| **headyconnection.org** | Community & nonprofit | Cloudflare Pages (static) | None (public) |
| **headybuddy.org** | HeadyBuddy extension landing | Cloudflare Pages (static) | None (public) |
| **headymcp.com** | MCP protocol docs & SSE endpoint | Cloud Run → MCP bridge | API key (Bearer) |
| **headyio.com** | I/O & webhook ingress | Cloudflare Worker → Cloud Run | HMAC signature |
| **headybot.com** | Bot / chat embed | Cloudflare Worker | Session token |
| **headyos.com** | Latent OS dashboard | Cloud Run | OAuth |
| **headyapi.com** | Public REST API gateway | Cloud Run (`heady-manager`) | API key (Bearer) |
| **headyapp.com** | Web application (HeadyWeb) | Cloud Run / Vercel | OAuth + session |
| **headylens.com** | Telemetry & observability | Cloud Run | API key |
| **heady.headyme.com** | MCP SSE server (IDE integration) | Cloud Run (`heady-manager`) | API key (Bearer) |

### Routing Rules

1. **Edge Workers** (`heady-edge-node`) run on Cloudflare for all 12+ domains. They handle:
   - Origin routing based on `Host` header
   - CORS enforcement (explicit origin allowlist — **no wildcards**)
   - Rate-limit enforcement (token-bucket per API key tier)
   - Request signing for internal origins

2. **Static sites** (headysystems.com, headyconnection.org, headybuddy.org) are served directly from Cloudflare Pages with no backend origin.

3. **Dynamic sites** route through the edge worker to GCP Cloud Run, which hosts `heady-manager` as the unified backend.

---

## Origin Map

```
┌─────────────────────────────────────┐
│  Cloudflare Edge (all 12 domains)   │
│  heady-edge-node Worker             │
│  ├─ DNS + SSL termination           │
│  ├─ CORS allowlist enforcement      │
│  ├─ Rate limiting (per-tier)        │
│  └─ Request signing                 │
└──────────────┬──────────────────────┘
               │
       ┌───────┴───────┐
       │               │
  Static Sites    Dynamic Origins
  (CF Pages)      (Cloud Run)
  ┌──────────┐    ┌──────────────────┐
  │ systems  │    │ heady-manager    │
  │ connect  │    │  ├─ /api/*       │
  │ buddy    │    │  ├─ /auth/*      │
  └──────────┘    │  ├─ /mcp/sse     │
                  │  └─ /health      │
                  └──────────────────┘
```

---

## Environment Variables (Domain Config)

Domain routing configuration is driven by environment variables set on the edge worker and on `heady-manager`. There is **no hardcoded domain list** in application code.

| Variable | Where Set | Purpose |
|---|---|---|
| `ALLOWED_ORIGINS` | Edge worker + Cloud Run | Comma-separated allowlist for CORS |
| `PRIMARY_DOMAIN` | Cloud Run | `headyme.com` — used for OAuth redirect base |
| `MCP_SSE_ORIGIN` | Cloud Run | `heady.headyme.com` — MCP SSE endpoint |
| `API_DOMAIN` | Cloud Run | `headyapi.com` — public API base |
| `CLOUDFLARE_ZONE_IDS` | Edge worker | Per-domain Cloudflare zone identifiers |

### Security Posture

- **No wildcard CORS.** Every allowed origin is enumerated explicitly.
- **Secrets are env-only.** No domain config, API keys, or tokens in source code.
- **mTLS between edge and origin.** Cloudflare authenticated origin pulls ensure only the edge can reach Cloud Run.

---

## Consuming the Registry

Any service that needs to resolve a Heady domain should read from environment variables, not from a hardcoded map. The canonical order of precedence:

1. `process.env.ALLOWED_ORIGINS` for CORS decisions
2. `process.env.PRIMARY_DOMAIN` for constructing redirect URLs
3. Cloudflare Worker bindings for edge-specific routing

This ensures consistency across `heady-manager`, edge workers, HeadyWeb, and any future surface.

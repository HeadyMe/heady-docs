# Heady Systems — API Keys & Services Reference

> Last updated: 2026-03-06 | Source: `.env` (not committed)

> **SECURITY NOTE:** API keys and secrets must NEVER be committed to this repository.
> All keys are stored in `.env` files (gitignored) or via a secrets manager (1Password Service Accounts).

---

## Service Inventory (14 services)

### Active & Verified (12 services)

| Env Var | Service | Description |
|---|---|---|
| `DATABASE_URL` | **Neon Postgres (Scale)** | PG 16, pgvector, 5 tables, 19 indexes |
| `NEON_API_KEY` | **Neon Management** | Projects, branches, autoscaling |
| `PERPLEXITY_API_KEY` | **Perplexity Sonar Pro** | Deep research with citations |
| `GROQ_API_KEY` | **Groq** | Ultra-fast LLM inference |
| `OPENAI_API_KEY` | **OpenAI** | GPT-4o, service account |
| `HF_TOKEN` | **Hugging Face** | Embeddings, HeadyVinci |
| `GEMINI_API_KEY` | **Google Gemini** | Gemini + 6 additional Google keys |
| `GITHUB_TOKEN` | **GitHub** | Repository access (HeadyConnection) |
| `CLOUDFLARE_API_TOKEN` | **Cloudflare** | Edge Workers, DNS, CDN |
| `STRIPE_SECRET_KEY` | **Stripe (Live)** | Payment processing |
| `PINECONE_API_KEY` | **Pinecone** | Distributed vector DB |
| `SENTRY_DSN` | **Sentry** | Error tracking (project: heady-manager) |

### Requires Attention (2 services)

| Env Var | Service | Issue | Fix |
|---|---|---|---|
| `CLAUDE_API_KEY` | **Anthropic** | Key valid, no credits | Top up at anthropic.com/billing |
| `UPSTASH_REDIS_REST_URL` | **Upstash** | Missing REST URL | Get from console.upstash.com |

---

## Required Environment Variables

### AI Providers (10 keys)

| Var | Service |
|---|---|
| `PERPLEXITY_API_KEY` | Perplexity Sonar Pro |
| `GROQ_API_KEY` | Groq fast LLM |
| `OPENAI_API_KEY` | OpenAI GPT-4o |
| `CLAUDE_API_KEY` | Anthropic Claude |
| `ANTHROPIC_ADMIN_KEY` | Anthropic admin |
| `ANTHROPIC_ORG_ID` | Anthropic org |
| `HF_TOKEN` | Hugging Face |
| `GEMINI_API_KEY` | Gemini (heady project) |
| `GEMINI_API_KEY_HEADY` | Gemini (colab) |
| `GOOGLE_API_KEY` | Google Cloud default |

Additional Google keys: `GOOGLE_API_KEY_SECONDARY`, `GCLOUD_API_KEY`, `GOOGLE_CLOUD_API_KEY`, `FIREBASE_API_KEY`

### DevOps & Infrastructure (9 keys)

| Var | Service |
|---|---|
| `GITHUB_TOKEN` | GitHub primary |
| `GITHUB_TOKEN_SECONDARY` | GitHub secondary |
| `CLOUDFLARE_API_TOKEN` | Cloudflare primary |
| `CLOUDFLARE_API_TOKEN_2` | Cloudflare secondary |
| `CLOUDFLARE_API_TOKEN_3` | Cloudflare tertiary |
| `SENTRY_AUTH_TOKEN` | Sentry org token |
| `SENTRY_PERSONAL_TOKEN` | Sentry personal |
| `SENTRY_DSN` | Sentry DSN |
| `OP_SERVICE_ACCOUNT_TOKEN` | 1Password Service Account |

### Database & Cache (4 keys)

| Var | Service |
|---|---|
| `DATABASE_URL` | Neon Postgres Scale |
| `NEON_API_KEY` | Neon Management |
| `PINECONE_API_KEY` | Pinecone vectors |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis |

### Commerce (1 key)

| Var | Service |
|---|---|
| `STRIPE_SECRET_KEY` | Stripe Live payments |

### Heady Internal (2 keys)

| Var | Service |
|---|---|
| `HEADY_API_KEY` | API gateway auth |
| `ADMIN_TOKEN` | Admin access |

---

## Setup Instructions

1. Copy `.env.example` to `.env` (if available) or create `.env` from the table above
2. Populate each key from the respective service dashboard
3. For 1Password-managed secrets, use: `op run --env-file=.env -- <command>`
4. Never commit `.env` files — ensure `.gitignore` includes `.env*`

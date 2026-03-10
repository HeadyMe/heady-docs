# Heady Systems — API Keys & Services Reference

> Last updated: 2026-03-06 | Source: `.env`

---

## Overview

Heady integrates with multiple external services. This document lists required environment variables by category. For setup instructions, see the [Authentication Guide](../docs/reference/authentication.md).

> **Note:** Never commit API keys or secrets to source control. Use environment variables, `.env` files (gitignored), or a secret manager such as 1Password Service Accounts.

---

## ✅ Active & Verified Services (12)

| Env Var | Service | Status | Description |
|---|---|---|---|
| `DATABASE_URL` | **Neon Postgres (Scale)** | ✅ Connected | PG 16, pgvector, 5 tables, 19 indexes |
| `NEON_API_KEY` | **Neon Management** | ✅ Active | Projects, branches, autoscaling |
| `PERPLEXITY_API_KEY` | **Perplexity Sonar Pro** | ✅ Active | Deep research with citations |
| `GROQ_API_KEY` | **Groq** | ✅ Active | Ultra-fast LLM inference |
| `OPENAI_API_KEY` | **OpenAI** | ✅ Active | GPT-4o, service account |
| `HF_TOKEN` | **Hugging Face** | ✅ Active | Embeddings, HeadyVinci |
| `GEMINI_API_KEY` | **Google Gemini** | ✅ Active | Multi-model inference (+ additional Google keys) |
| `GITHUB_TOKEN` | **GitHub** | ✅ Active | Repository access |
| `CLOUDFLARE_API_TOKEN` | **Cloudflare** | ✅ Active | Edge workers, DNS, CDN |
| `STRIPE_SECRET_KEY` | **Stripe** | ✅ Active | Payment processing |
| `PINECONE_API_KEY` | **Pinecone** | ✅ Active | Distributed vector DB |
| `SENTRY_DSN` | **Sentry** | ✅ Active | Error tracking |

## ⚠️ Services Requiring Attention (2)

| Env Var | Service | Issue | Resolution |
|---|---|---|---|
| `CLAUDE_API_KEY` | **Anthropic** | No credits remaining | Top up via Anthropic billing dashboard |
| `UPSTASH_REDIS_REST_URL` | **Upstash** | URL not configured | Retrieve from Upstash console > REST API section |

---

## Environment Variables by Category

### AI Providers (10 keys)

| Var | Service | Notes |
|---|---|---|
| `PERPLEXITY_API_KEY` | Perplexity Sonar Pro | Deep research API |
| `GROQ_API_KEY` | Groq | Fast LLM inference |
| `OPENAI_API_KEY` | OpenAI | GPT-4o service account |
| `CLAUDE_API_KEY` | Anthropic Claude | Primary Claude access |
| `ANTHROPIC_ADMIN_KEY` | Anthropic | Admin operations |
| `ANTHROPIC_ORG_ID` | Anthropic | Organization identifier |
| `HF_TOKEN` | Hugging Face | Embeddings and model access |
| `GEMINI_API_KEY` | Google Gemini | Primary Gemini project |
| `GEMINI_API_KEY_HEADY` | Google Gemini | Colab cluster access |
| `GOOGLE_API_KEY` | Google Cloud | Default cloud services |

Additional Google keys: `GOOGLE_API_KEY_SECONDARY`, `GCLOUD_API_KEY`, `GOOGLE_CLOUD_API_KEY`, `FIREBASE_API_KEY`

### DevOps & Infrastructure (10 keys)

| Var | Service | Notes |
|---|---|---|
| `GITHUB_TOKEN` | GitHub | Primary repository access |
| `GITHUB_TOKEN_SECONDARY` | GitHub | Secondary / CI access |
| `CLOUDFLARE_API_TOKEN` | Cloudflare | Primary edge workers |
| `CLOUDFLARE_API_TOKEN_2` | Cloudflare | Secondary workers |
| `CLOUDFLARE_API_TOKEN_3` | Cloudflare | Tertiary workers |
| `SENTRY_AUTH_TOKEN` | Sentry | Organization-level token |
| `SENTRY_PERSONAL_TOKEN` | Sentry | Personal access token |
| `SENTRY_DSN` | Sentry | Error tracking DSN |
| `OP_SERVICE_ACCOUNT_TOKEN` | 1Password | Service account for secret management |

### Database & Cache (4 keys)

| Var | Service | Notes |
|---|---|---|
| `DATABASE_URL` | Neon Postgres | Primary database (Scale plan) |
| `NEON_API_KEY` | Neon | Management API access |
| `PINECONE_API_KEY` | Pinecone | Vector database |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis | Cache layer (requires URL configuration) |

### Commerce (1 key)

| Var | Service | Notes |
|---|---|---|
| `STRIPE_SECRET_KEY` | Stripe | Live payment processing |

### Heady Internal (2 keys)

| Var | Service | Notes |
|---|---|---|
| `HEADY_API_KEY` | Heady Gateway | API gateway authentication |
| `ADMIN_TOKEN` | Heady Admin | Administrative access |

---

## Setup Instructions

1. Copy the `.env.example` file (if available) or create a new `.env` in the project root
2. Populate each variable with your credentials
3. Ensure `.env` is listed in `.gitignore`
4. For team environments, use 1Password Service Accounts (`OP_SERVICE_ACCOUNT_TOKEN`) to inject secrets at runtime

For authentication flows and token management, see the [Authentication Guide](../docs/reference/authentication.md).

---

*© 2026 Heady Systems LLC. All rights reserved.*

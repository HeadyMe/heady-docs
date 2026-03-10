# Authentication & Authorization

> How authentication works across the Heady platform.

---

## Overview

Heady uses a **defense-in-depth** authentication model with multiple methods depending on the access pattern:

| Method | Use Case | Layer |
|--------|----------|-------|
| **API Keys** | MCP tool access, programmatic integration | Gateway |
| **OAuth 2.0** | Web application login, third-party integrations | Gateway |
| **Mutual TLS (mTLS)** | Inter-service communication | Gateway |
| **Scoped Personal Access Tokens** | Git operations, CI/CD pipelines | Gateway |
| **Subscription Tiers** | Rate limiting and feature gating | Orchestration |

---

## API Key Authentication

API keys are the primary method for accessing Heady MCP tools from IDEs and programmatic clients.

### Obtaining an API Key

1. Sign in to the Heady admin dashboard
2. Navigate to **Settings > API Keys**
3. Click **Generate New Key**
4. Select the required scopes (see [Scopes](#scopes) below)
5. Copy the key — it will only be shown once

### Using Your API Key

Include your key in the `Authorization` header:

```
Authorization: Bearer <your-api-key>
```

Or set it as an environment variable for MCP integration:

```bash
export HEADY_API_KEY="<your-api-key>"
```

### Key Management Best Practices

- **Rotate keys regularly** — set a rotation schedule (recommended: every 90 days)
- **Use scoped keys** — create keys with the minimum required permissions
- **Never commit keys to source control** — use environment variables or a secret manager (e.g., 1Password Service Accounts)
- **Monitor usage** — review key activity in the admin dashboard for anomalies
- **Revoke compromised keys immediately** — generate a replacement before revoking if needed for continuity

---

## OAuth 2.0 Flow

Heady supports OAuth 2.0 for web application authentication across its 12+ managed domains.

### Authorization Code Flow

```
1. Client redirects user to:
   https://headyme.com/oauth/authorize?
     client_id=<CLIENT_ID>&
     redirect_uri=<CALLBACK_URL>&
     response_type=code&
     scope=<SCOPES>

2. User authenticates on headyme.com

3. Heady redirects back with authorization code:
   <CALLBACK_URL>?code=<AUTH_CODE>

4. Client exchanges code for access token:
   POST https://headyme.com/oauth/token
   {
     "grant_type": "authorization_code",
     "code": "<AUTH_CODE>",
     "client_id": "<CLIENT_ID>",
     "client_secret": "<CLIENT_SECRET>",
     "redirect_uri": "<CALLBACK_URL>"
   }

5. Response includes access_token and refresh_token
```

### Cross-Domain Authentication

Heady manages 12+ domains (headyme.com, headysystems.com, headyconnection.org, headybuddy.org, headymcp.com, headyio.com, headybot.com, headyos.com, headyapi.com, headyapp.com, headylens.com, and others). Authentication is unified across all domains via the Cloudflare Edge Layer:

- Single sign-on is handled at the edge via shared session tokens
- All domains route through the same Cloudflare Workers origin (`heady-edge-node`)
- Session state is stored in the vector memory layer, not cookies

---

## Scopes

API keys and OAuth tokens are scoped to specific permissions:

| Scope | Access Granted |
|-------|---------------|
| `chat` | HeadyBuddy, HeadyChat, HeadySoul |
| `dev` | HeadyCoder, HeadyCodex, HeadyCopilot, HeadyRefactor, HeadyAnalyze |
| `research` | HeadyResearch, HeadyRisks, HeadyLens |
| `memory` | HeadyMemory, HeadyEmbed, HeadyVinci, HeadyDeepScan |
| `creative` | HeadyDesign, HeadyCanvas, HeadyMedia |
| `ops` | HeadyDeploy, HeadyOps, HeadyHealth, HeadyMaid, HeadyMaintenance |
| `quality` | HeadyBattle, HeadyAutoFlow, HeadyDoctor |
| `admin` | Full platform access, key management, user administration |

Request only the scopes your integration requires. Tokens with `admin` scope should be limited to administrative tooling.

---

## Subscription Tiers & Rate Limiting

Heady uses a 9-tier subscription system that controls both feature access and rate limits:

| Tier | Rate Limit | Features |
|------|------------|----------|
| Free | 10 req/min | Chat, basic research |
| Starter | 60 req/min | + Dev tools, memory |
| Pro | 300 req/min | + Creative, ops |
| Enterprise | Custom | Full access, SLA, dedicated support |

Rate limit headers are included in all API responses:

```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 287
X-RateLimit-Reset: 1710100000
```

When rate limited, the API returns `429 Too Many Requests` with a `Retry-After` header.

---

## Token Lifecycle

| Event | Behavior |
|-------|----------|
| **Issuance** | Tokens are issued with a configurable TTL (default: 24 hours for access tokens, 30 days for refresh tokens) |
| **Refresh** | Use the refresh token to obtain a new access token without re-authentication |
| **Expiration** | Expired tokens return `401 Unauthorized`. Use the refresh flow to re-authenticate. |
| **Revocation** | Tokens can be revoked via the admin dashboard or the `/oauth/revoke` endpoint |
| **Rotation** | API keys should be rotated on a regular schedule. The admin dashboard supports seamless rotation with overlap periods. |

---

## Security Architecture

For full details on Heady's security philosophy, including post-quantum cryptography readiness, SHA-256 audit trails, and zero-trust CI/CD, see the [Architecture & Design Patterns](../sources/05-heady-architecture-and-patterns.md#security-philosophy) documentation.

---

*© 2026 Heady Systems LLC. All rights reserved.*

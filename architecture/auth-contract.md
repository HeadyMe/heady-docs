# Cross-Site Authentication Contract

> Heady Systems LLC — Auth model, token lifecycle, and implementation scaffolding for all surfaces.

---

## Auth Methods

Heady supports three authentication methods, each scoped to specific surfaces:

| Method | Surface | Token Type | Lifetime | Storage |
|---|---|---|---|---|
| **API Key** | REST API, MCP SSE, CLI | Bearer token (`HEADY_API_KEY`) | Long-lived (rotate quarterly) | `Authorization: Bearer <key>` header |
| **OAuth 2.0 (PKCE)** | HeadyWeb, headyme.com portal, HeadyOS dashboard | JWT access + refresh | Access: 15 min / Refresh: 7 days | `httpOnly` secure cookie |
| **HMAC Signature** | Webhooks, headyio.com ingress | Request signature | Per-request | `X-Heady-Signature` header |

---

## API Key Authentication

Used by IDE integrations, CLI tools, and programmatic access.

### Key Format

```
heady_api_key_<version>_<random>
```

- Version `001` = standard user key
- Version `002` = admin key (elevated privileges)
- Keys are generated at **headyme.com** → Settings → API Keys

### Request Flow

```
Client                     Edge Worker              heady-manager
  │                            │                         │
  ├─ Authorization: Bearer k ──►                         │
  │                            ├─ Validate key format    │
  │                            ├─ Rate-limit check       │
  │                            ├─ Forward + sign ────────►
  │                            │                         ├─ DB lookup (keys table)
  │                            │                         ├─ Check tier + scopes
  │                            │                         ├─ Return response
  │                            ◄─────────────────────────┤
  ◄────────────────────────────┤
```

### Rate Limiting by Tier

| Tier | Requests/min | Concurrent | Scopes |
|---|---|---|---|
| Free | 20 | 2 | read-only |
| Pro | 200 | 10 | read + write |
| Enterprise | 2000 | 100 | full access |
| Admin | Unlimited | Unlimited | full + system |

---

## OAuth 2.0 (PKCE) Authentication

Used by browser-based surfaces (HeadyWeb, headyme.com portal).

### Authorization Flow

```
Browser                    headyme.com/auth          heady-manager
  │                            │                         │
  ├─ GET /auth/login ──────────►                         │
  │  (PKCE code_challenge)     │                         │
  │                            ├─ Redirect to provider   │
  │                            │  (Google / GitHub)      │
  │  ◄── Callback with code ───┤                         │
  │                            ├─ Exchange code ─────────►
  │                            │                         ├─ Verify + issue JWT
  │                            │                         ├─ Set httpOnly cookie
  │  ◄── Set-Cookie: token ────┤◄────────────────────────┤
  │                            │                         │
  ├─ Subsequent requests ──────►                         │
  │  (Cookie sent auto)        ├─ Validate JWT ──────────►
  │                            │                         ├─ Check exp + scopes
```

### Token Structure

```json
{
  "sub": "user_abc123",
  "iss": "headyme.com",
  "aud": ["headyapi.com", "headyapp.com", "headyos.com"],
  "tier": "pro",
  "scopes": ["read", "write", "mcp"],
  "exp": 1741700000,
  "iat": 1741699100
}
```

### Cookie Policy

| Attribute | Value | Rationale |
|---|---|---|
| `httpOnly` | `true` | Prevent XSS token theft |
| `secure` | `true` | HTTPS only |
| `sameSite` | `Strict` | Prevent CSRF |
| `domain` | `.headyme.com` | Shared across subdomains of primary domain |
| `path` | `/` | Available to all routes |
| `maxAge` | `604800` (7 days) | Refresh token lifetime |

### Cross-Domain Session Sharing

Since Heady operates across multiple TLDs (headyme.com, headyapi.com, headyapp.com), the cookie is scoped to `.headyme.com` only. Cross-domain requests to `headyapi.com` use the **API key** method, not cookies.

For HeadyWeb on `headyapp.com`:
1. Initial login redirects to `headyme.com/auth/login`
2. After OAuth, redirect back to `headyapp.com/callback` with a one-time code
3. HeadyWeb exchanges the code for its own session token via `headyapi.com`

---

## HMAC Webhook Authentication

Used for inbound webhooks on `headyio.com`.

### Signature Verification

```
signature = HMAC-SHA256(webhook_secret, timestamp + "." + body)
```

Headers:
- `X-Heady-Signature`: hex-encoded HMAC
- `X-Heady-Timestamp`: Unix epoch seconds

Verification steps:
1. Reject if timestamp is older than 5 minutes (replay protection)
2. Recompute HMAC with stored `webhook_secret`
3. Constant-time comparison of signatures

---

## Implementation Scaffolding

### Middleware Pattern (heady-manager)

```javascript
// auth-middleware.js — route-level auth selection
const AUTH_METHODS = {
  apiKey:  requireApiKey,   // checks Authorization header
  oauth:   requireOAuth,    // validates JWT from cookie
  hmac:    requireHMAC,     // verifies webhook signature
  public:  passthrough,     // no auth required
};

function authMiddleware(method) {
  return (req, res, next) => {
    const handler = AUTH_METHODS[method];
    if (!handler) return res.status(500).json({ error: 'unknown auth method' });
    return handler(req, res, next);
  };
}

// Route registration
app.use('/api/v1/*',       authMiddleware('apiKey'));
app.use('/mcp/sse',        authMiddleware('apiKey'));
app.use('/auth/*',         authMiddleware('public'));
app.use('/webhooks/*',     authMiddleware('hmac'));
app.use('/dashboard/*',    authMiddleware('oauth'));
```

### Environment Variables

| Variable | Service | Purpose |
|---|---|---|
| `JWT_SECRET` | heady-manager | Signs OAuth JWTs |
| `OAUTH_CLIENT_ID` | heady-manager | OAuth provider client ID |
| `OAUTH_CLIENT_SECRET` | heady-manager | OAuth provider secret |
| `WEBHOOK_SECRET` | heady-manager | HMAC key for webhook verification |
| `ALLOWED_ORIGINS` | Edge + heady-manager | CORS allowlist (explicit, no wildcards) |

### Key Rotation

- API keys: rotate quarterly via headyme.com admin panel
- JWT secret: rotate with zero-downtime by supporting `JWT_SECRET` + `JWT_SECRET_PREVIOUS` during rollover
- Webhook secrets: rotate per-integration via API

---

## Security Invariants

1. **Secrets are env-only.** Never in source code, never in git history.
2. **No wildcard CORS.** Every origin is explicitly listed in `ALLOWED_ORIGINS`.
3. **All tokens expire.** API keys have optional expiry; JWTs always expire.
4. **mTLS between services.** Edge-to-origin and inter-node communication use mutual TLS.
5. **Audit trail.** Every auth event (login, key use, failure) is logged to the structured logger with `SHA-256` action hashes.

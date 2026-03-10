# Getting Started with Heady

> Quickstart guide for developers and technical evaluators.

---

## Prerequisites

Before connecting to the Heady platform, ensure you have:

- **Node.js** v18+ and npm/pnpm
- **Git** configured with a GitHub account
- A compatible IDE with MCP support (e.g., VS Code with Claude extension, Cursor, or Antigravity IDE)
- Access credentials (API key or OAuth token) — see [Authentication](../reference/authentication.md)

---

## 5-Step Quickstart

### Step 1: Authenticate

Obtain your Heady API key from the admin dashboard or your organization administrator.

```bash
# Set your API key in the environment
export HEADY_API_KEY="your-api-key-here"
```

Alternatively, create a `.env` file in your project root:

```env
HEADY_API_KEY=your-api-key-here
```

> See [Authentication](../reference/authentication.md) for full details on API keys, OAuth, and cross-domain auth.

### Step 2: Configure Your IDE for MCP

Heady's 30+ tools are accessible through the **Model Context Protocol (MCP)**. Connect your IDE to the MCP server:

**For Claude Desktop / VS Code with Claude:**

Add to your MCP configuration (`claude_desktop_config.json` or equivalent):

```json
{
  "mcpServers": {
    "heady": {
      "url": "https://heady.headyme.com/sse",
      "headers": {
        "Authorization": "Bearer ${HEADY_API_KEY}"
      }
    }
  }
}
```

**For Cursor:**

Add the same MCP server configuration via Cursor Settings > MCP Servers.

### Step 3: Verify Connection

Once configured, verify your MCP connection is active:

1. Open your IDE's MCP tool panel
2. You should see Heady tools listed (e.g., `heady-chat`, `heady-coder`, `heady-memory`)
3. Run a test command:

```
Use the heady-health tool to check system status
```

You should receive a status response confirming connectivity to the Heady platform.

### Step 4: Explore Available Tools

Heady provides 30+ MCP tools organized into 7 domains:

| Domain | Key Tools | What They Do |
|--------|-----------|--------------|
| **Chat** | HeadyBuddy, HeadyChat | Conversational AI with persistent memory |
| **Dev Tools** | HeadyCoder, HeadyCodex, HeadyCopilot | Code generation, analysis, refactoring |
| **Research** | HeadyResearch, HeadyRisks | Web research with citations, security scanning |
| **Memory** | HeadyMemory, HeadyEmbed | 3D vector memory — store and recall across sessions |
| **Creative** | HeadyDesign, HeadyCanvas | UI/UX generation with Sacred Geometry principles |
| **Ops** | HeadyDeploy, HeadyHealth | Deployment, monitoring, maintenance |
| **Quality** | HeadyBattle, HeadyAutoFlow | Competitive AI evaluation, automated pipelines |

> See the full [Service Catalog](../sources/04-heady-service-catalog-and-capabilities.md) for details on all tools.

### Step 5: Create Your First HeadyBee

A **HeadyBee** is the atomic unit of work in the Heady ecosystem. Each bee is a focused task agent.

To spawn a bee through the MCP interface:

```
Use heady-coder to analyze this repository and suggest improvements
```

The platform will:
1. Create a HeadyBee scoped to the task
2. Execute the work using the appropriate AI model(s)
3. Return structured results with citations and confidence scores
4. Store the interaction in vector memory for future context

---

## What's Next?

- **[Authentication Guide](../reference/authentication.md)** — Deep dive into API keys, OAuth, tokens, and cross-domain auth
- **[Architecture Overview](../sources/05-heady-architecture-and-patterns.md)** — Understand the six-layer stack and design patterns
- **[Service Catalog](../sources/04-heady-service-catalog-and-capabilities.md)** — Full reference for all 30+ MCP tools
- **[Glossary](../reference/glossary.md)** — Definitions for Heady-specific terminology

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| MCP connection timeout | Verify your API key is valid and not expired. Check network access to `heady.headyme.com`. |
| Tools not appearing in IDE | Restart your IDE after adding MCP configuration. Ensure the config file path is correct. |
| Authentication error | Confirm your API key has the required scopes. See [Authentication](../reference/authentication.md). |
| Rate limit exceeded | Check your subscription tier's rate limits. Upgrade or implement request throttling. |

---

*© 2026 Heady Systems LLC. All rights reserved.*

---
name: mixrpay
description: Agent payment infrastructure. Budget-controlled LLM calls, MCP server deployment, and nested agent spawning.
license: MIT
metadata:
  author: mixrpay
  version: "2.2.0"
  sdk_version: "0.8.8"
  repository: https://github.com/mixrpay/clawhub-skill-mixrpay
compatibility: Requires node>=18. Set MIXRPAY_SESSION_KEY env var.
---

# MixrPay - Budget-Controlled Payments for AI Agents

Give your agent a wallet with spending limits. Make LLM calls. Deploy MCP servers. Spawn child agents.

## When to Use

- Making LLM calls with budget controls
- Deploying MCP servers with your own API keys
- Spawning child agents for parallel work
- Human says "budget", "spending", "deploy server", "spawn agent"

## Quick Start

### 1. Initialize (Session Key Pre-Configured)

Your session key is already set in `MIXRPAY_SESSION_KEY` when deployed via MixrPay.

```javascript
import { AgentWallet } from '@mixrpay/agent-sdk';

const wallet = new AgentWallet({
  sessionKey: process.env.MIXRPAY_SESSION_KEY
});
```

### 2. Make LLM Calls

```javascript
// Simple completion (recommended for basic tasks)
const result = await wallet.complete('Summarize this article: ...');
console.log(result.text);
console.log(`Cost: $${result.costUsd.toFixed(4)}`);

// With options
const result = await wallet.complete('Write a haiku', {
  model: 'gpt-4o',
  systemPrompt: 'You are a poet.'
});
```

### 3. Check Budget

```javascript
const stats = await wallet.getSpendingStats();
console.log(`Spent: $${stats.totalSpentUsd}`);
console.log(`Remaining: $${stats.remainingTotalUsd ?? 'unlimited'}`);

if (stats.remainingTotalUsd !== null && stats.remainingTotalUsd < estimatedCost) {
  return "Need more budget from human";
}
```

### 4. Claim an Invite (Manual Setup)

If not auto-provisioned, an agent can claim an invite manually:

```javascript
const result = await AgentWallet.claimInvite({
  inviteCode: 'mixr-abc123...',
  privateKey: '0xYOUR_PRIVATE_KEY'
});

// Save the session key
process.env.MIXRPAY_SESSION_KEY = result.sessionKey;
```

## Core Methods

| Method | Purpose |
|--------|---------|
| `wallet.complete(prompt, options?)` | Simple LLM call |
| `wallet.runAgent(options)` | Multi-turn with tools |
| `wallet.getSpendingStats()` | Check budget |
| `wallet.getAvailableBudget()` | Detailed budget breakdown |
| `wallet.spawnChildInvite(options)` | Create child agent |
| `wallet.getChildSessions()` | List child agents |
| `wallet.deployJitMcp(options)` | Deploy MCP server |
| `wallet.listJitInstances()` | List deployed servers |
| `wallet.stopJitInstance(id)` | Stop a server |
| `wallet.searchGlamaDirectory(query)` | Search for MCP servers |
| `wallet.fetch(url, options)` | HTTP with auto x402 payment |
| `wallet.getBalance()` | Wallet USDC balance |
| `wallet.getSessionKeyInfo()` | Session key details |
| `wallet.runDiagnostics()` | Health check |

## Nested Agent Spawning

Delegate budget to child agents (max 20% of available):

```javascript
// Check what's available first
const budget = await wallet.getAvailableBudget();
console.log(`Can spawn up to: $${budget.maxSpawnBudget}`);

if (budget.canSpawn) {
  const child = await wallet.spawnChildInvite({
    budgetUsd: 10.00,
    name: 'Research Agent',
    expiresInDays: 7
  });

  // Share child.inviteCode with child process
  console.log(`Child invite: ${child.inviteCode}`);
}

// List children
const children = await wallet.getChildSessions();
children.forEach(c => console.log(`${c.name}: $${c.spentUsd}/$${c.budgetUsd}`));
```

## Deploy MCP Servers

Deploy any MCP server with your own API keys:

```javascript
// Search for servers
const results = await wallet.searchGlamaDirectory('notion');
console.log(results.servers.map(s => `${s.id}: ${s.name}`));

// Deploy
const instance = await wallet.deployJitMcp({
  glamaId: 'notion-mcp',
  glamaNamespace: 'notion',
  glamaSlug: 'notion-mcp',
  toolName: 'My Notion',
  envVars: { NOTION_API_KEY: 'secret_...' },
  ttlHours: 24
});

// Use instance.instance.endpointUrl

// List active instances
const instances = await wallet.listJitInstances({ status: 'active' });

// Stop when done
await wallet.stopJitInstance(instance.instance.id);
```

## Error Handling

```javascript
try {
  const result = await wallet.complete('...');
} catch (error) {
  if (error.code === 'BUDGET_EXHAUSTED') {
    return "Out of budget. Ask human for more.";
  }
  if (error.code === 'SESSION_EXPIRED') {
    return "Session expired. Need new invite.";
  }
  throw error;
}
```

## Available Models

- `gpt-4o-mini` (default, fast & cheap)
- `gpt-4o` (complex reasoning)
- `claude-sonnet-4-5` (long context)
- `claude-haiku-4-5` (fast)

## File Sharing

Share workspace files with your owner via signed download links. Links are secure (HMAC-signed) and expire after 24 hours by default.

### Generate a Download Link

```bash
curl -s -X POST http://localhost:8080/workspace/share \
  -H "Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"path": "my-report.md"}'
```

**Request body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Relative path within workspace (e.g., `report.md` or `docs/plan.md`) |
| `expiresIn` | number | No | Link lifetime in seconds (default: 86400 = 24h, max: 604800 = 7d) |

**Response:**
```json
{
  "url": "https://agent-xxx.up.railway.app/workspace/files/my-report.md?sig=abc123&exp=1700000000",
  "filename": "my-report.md",
  "size": 4096,
  "expiresAt": "2025-11-15T00:00:00.000Z"
}
```

### Example Workflow

1. Write a file with the `write` tool
2. Generate a signed URL with `bash` + `curl` (as above)
3. Send the `url` from the response to your owner in chat

### Notes

- Links expire after 24 hours. Generate a new one if the owner needs it again.
- Credential files (`.env`, `.key`, `.pem`, `.token`, `.secret`, dotfiles) cannot be shared.
- Maximum file size: 50MB.

## Links

- **Dashboard**: https://mixrpay.com/wallet
- **SDK Docs**: https://mixrpay.com/docs/sdk
- **npm**: `npm install @mixrpay/agent-sdk`

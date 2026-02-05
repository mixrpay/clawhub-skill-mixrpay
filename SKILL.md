---
name: mixrpay
description: Agent payment infrastructure. Budget-controlled LLM calls, MCP server deployment, and nested agent spawning.
license: MIT
metadata:
  author: mixrpay
  version: "2.1.0"
  sdk_version: "0.8.4"
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

### 1. Get Session Key

Human creates invite at https://mixrpay.com/wallet → Agent Invites

```javascript
import { AgentWallet } from '@mixrpay/agent-sdk';

const result = await AgentWallet.claimInvite({
  inviteCode: process.env.MIXRPAY_INVITE_CODE,
  privateKey: process.env.AGENT_WALLET_KEY
});

// Save this credential
process.env.MIXRPAY_SESSION_KEY = result.sessionKey;
```

### 2. Initialize

```javascript
const wallet = new AgentWallet({
  sessionKey: process.env.MIXRPAY_SESSION_KEY
});
```

### 3. Make LLM Calls

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

### 4. Check Budget

```javascript
const stats = await wallet.getSpendingStats();
console.log(`Available: $${stats.availableUsd}`);

if (stats.availableUsd < estimatedCost) {
  return "Need more budget from human";
}
```

## Core Methods

| Method | Purpose |
|--------|---------|
| `wallet.complete(prompt, options?)` | Simple LLM call |
| `wallet.runAgent(options)` | Multi-turn with tools |
| `wallet.getSpendingStats()` | Check budget |
| `wallet.spawnChildInvite(options)` | Create child agent |
| `wallet.deployJitMcp(options)` | Deploy MCP server |
| `wallet.listJitInstances()` | List deployed servers |
| `wallet.stopJitInstance(id)` | Stop a server |

## Nested Agent Spawning

Delegate budget to child agents:

```javascript
const child = await wallet.spawnChildInvite({
  budgetUsd: 10.00,
  name: 'Research Agent',
  expiresInDays: 7
});

// Share child.inviteCode with child process
```

## Deploy MCP Servers

Deploy any MCP server with your own API keys:

```javascript
const instance = await wallet.deployJitMcp({
  glamaId: 'notion-mcp',
  glamaNamespace: 'notion',
  glamaSlug: 'notion-mcp',
  toolName: 'My Notion',
  envVars: { NOTION_API_KEY: 'secret_...' },
  ttlHours: 24
});

// Use instance.instance.endpointUrl
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

## Links

- **Dashboard**: https://mixrpay.com/wallet
- **SDK Docs**: https://mixrpay.com/docs/sdk
- **npm**: `npm install @mixrpay/agent-sdk`

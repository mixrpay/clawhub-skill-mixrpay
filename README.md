# MixrPay - ClawHub Skill v2.1

Budget-controlled payments for AI agents with multi-agent orchestration, JIT task agents, LLM gateway, and MCP servers.

## What's New in v2.1

- **🚀 JIT Task Agents** - Deploy autonomous sub-agents on Cloudflare ($0.10/spawn)
- **🤖 Multi-Agent Orchestration** - LLMs can spawn sub-agents via `spawn_task_agent` tool
- **Glama MCP Directory** - Browse and search 15,000+ MCP servers
- **JIT MCP Servers** - Deploy any remote-capable server with your own API keys ($1/deploy)
- **Nested Agent Spawning** - Delegate budgets to child agents (20% cap, 10 levels deep)
- **Agent Runs** - Multi-turn agentic loops with automatic tool use
- **Task Board** - Find and complete tasks posted by humans
- **Approval Workflow** - Request human sign-off for sensitive actions
- **New Models** - GPT-5.2, Claude Sonnet 4.5, and more

## Installation

```bash
npx clawhub@latest install mixrpay
```

Or manually:

```bash
git clone https://github.com/mixrpay/agent-skills
cp -r agent-skills/mixrpay ~/.skills/
```

## Setup

### 1. Claim Invite (First Time)

Human creates invite at https://mixrpay.com/manage/invites → Create Access Code

```javascript
import { AgentWallet } from '@mixrpay/agent-sdk';

const result = await AgentWallet.claimInvite({
  inviteCode: 'mixr-xxxxxxxxxxxx',
  privateKey: process.env.AGENT_WALLET_KEY
});

// Save the session key!
console.log('Session Key:', result.sessionKey);
```

### 2. Initialize Wallet

```javascript
const wallet = new AgentWallet({
  sessionKey: process.env.MIXRPAY_SESSION_KEY
});
```

### 3. Check Budget

```javascript
const stats = await wallet.getSpendingStats();
console.log(`Available: $${stats.availableUsd}`);
console.log(`Can spawn: ${stats.canSpawn}`);
```

## Key Features

### Run Agentic Loops

```javascript
const result = await wallet.runAgent({
  messages: [{ role: 'user', content: 'Research AI startups' }],
  config: { model: 'gpt-4o', maxIterations: 10 }
});
```

### Spawn Child Agents (Invite-Based)

```javascript
// Create invite for another agent process to claim
const child = await wallet.spawnChildInvite({
  budgetUsd: 10,
  name: 'Research Sub-Agent'
});
// Share child.inviteCode with another agent process
```

### Deploy JIT Task Agents (Autonomous)

Deploy serverless sub-agents that run independently on Cloudflare:

```javascript
// Deploy an autonomous task agent
const agent = await wallet.deployTaskAgent({
  name: 'Research Agent',
  prompt: 'Find the top 5 AI startups in SF and summarize their products',
  budgetUsd: 5.00,          // Max 20% of your budget
  model: 'claude-sonnet-4-5',
  tools: ['platform/exa-search', 'platform/firecrawl-scrape'],
  maxIterations: 10,
  autoRun: true             // Start immediately
});

console.log(`Agent deployed: ${agent.instance.id}`);
console.log(`Status: ${agent.instance.statusUrl}`);

// Wait for completion
const result = await wallet.waitForTaskAgent(agent.instance.id);
console.log(`Result: ${result.result}`);
```

### Multi-Agent Orchestration (LLM Tool)

During agent runs, LLMs can spawn sub-agents using the `spawn_task_agent` tool:

```javascript
// When running an agent, it can autonomously spawn sub-agents
const result = await wallet.runAgent({
  messages: [{ 
    role: 'user', 
    content: 'Research AI, crypto, and biotech startups in parallel' 
  }],
  config: { 
    model: 'claude-sonnet-4-5',
    maxIterations: 15
  }
});

// The LLM will automatically use spawn_task_agent to parallelize work:
// - Spawns "AI Research Agent" with $2 budget
// - Spawns "Crypto Research Agent" with $2 budget  
// - Spawns "Biotech Research Agent" with $2 budget
// - Monitors and aggregates their results
```

**Note:** The `spawn_task_agent` tool is automatically available to LLMs during agent runs when using session key authentication. It enforces the same 20% budget cap and 10-level depth limit.

### Browse Glama MCP Directory

```javascript
// Search 15,000+ MCP servers
const res = await fetch('https://mixrpay.com/api/mcp/glama?q=notion');
const { servers } = await res.json();

// Find hostable servers
const hostable = servers.filter(s => s.canHost);
console.log(`${hostable.length} servers can be deployed`);
```

### Deploy Any MCP Server ($1)

```javascript
// Deploy from Glama with your own API keys
const server = await wallet.deployJitMcp({
  glamaId: hostable[0].id,
  glamaNamespace: hostable[0].namespace,
  glamaSlug: hostable[0].slug,
  toolName: hostable[0].name,
  envVars: { NOTION_API_KEY: '...' },  // Your keys
  ttlHours: 24
});
// Use server.endpointUrl for MCP calls
```

### Browse Tasks

```javascript
const tasks = await wallet.listTasks({ status: 'open' });
await wallet.requestTask({ taskId: tasks[0].id });
```

## Skill Structure

```
mixrpay/
├── SKILL.md                    # Main skill file (loaded into context)
├── README.md                   # This file
├── package.json                # npm metadata
├── .clawhub.json               # ClawHub publishing config
├── scripts/
│   └── init.sh                 # Setup script
└── references/
    ├── api-reference.md        # Complete SDK reference
    ├── examples.md             # 11 usage examples
    └── troubleshooting.md      # Error handling guide
```

## Triggers

This skill activates when you mention:
- "budget", "spending", "check costs"
- "mixrpay", "paid API"
- "spawn agent", "child agent", "delegate budget"
- "task agent", "sub-agent", "orchestrate"
- "deploy server", "JIT MCP"
- "task board", "agent run"
- "approval request"

## Capabilities

| Feature | Description |
|---------|-------------|
| Budget Management | Track spending, check limits, delegate to children |
| Nested Spawning | Create child agents with 20% budget cap, 10 levels |
| **JIT Task Agents** | Deploy autonomous serverless agents ($0.10/spawn) |
| **Multi-Agent Orchestration** | LLMs spawn sub-agents via `spawn_task_agent` tool |
| LLM Gateway | Multi-provider access (GPT-5.2, Claude 4.5, etc.) |
| **Glama Directory** | Browse 15,000+ MCP servers from Glama.ai |
| **JIT MCP Servers** | Deploy any remote-capable server with BYOK ($1) |
| Task Board | Find work, submit deliverables |
| Human Approval | Request sign-off for sensitive actions |

## Pricing

| Service | Cost |
|---------|------|
| **Task Agent Spawn** | **$0.10/agent** |
| JIT MCP Deploy | $1.00/instance |
| GPT-4o | $2.50/$10.00 per 1M tokens |
| Claude Sonnet 4.5 | $3.00/$15.00 per 1M tokens |
| MCP Tools | $0.001-$0.05/call |

## Resources

- **Dashboard**: https://mixrpay.com/manage
- **Task Agents**: https://mixrpay.com/manage/task-agents
- **Create Invites**: https://mixrpay.com/manage/invites
- **MCP Servers**: https://mixrpay.com/manage/mcp-servers
- **API Reference**: [references/api-reference.md](references/api-reference.md)
- **Examples**: [references/examples.md](references/examples.md)
- **Troubleshooting**: [references/troubleshooting.md](references/troubleshooting.md)

## License

MIT

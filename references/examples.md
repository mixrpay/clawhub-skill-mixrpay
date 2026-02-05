# MixrPay Examples

## Basic Setup

```javascript
import { AgentWallet } from '@mixrpay/agent-sdk';

const wallet = new AgentWallet({
  sessionKey: process.env.MIXRPAY_SESSION_KEY
});
```

## Simple LLM Calls

```javascript
// Basic
const result = await wallet.complete('What is 2+2?');
console.log(result.text);

// With options
const result = await wallet.complete('Explain AI', {
  model: 'gpt-4o',
  systemPrompt: 'Keep it simple.'
});
```

## Check Budget First

```javascript
const stats = await wallet.getSpendingStats();

if (stats.availableUsd < 5.00) {
  return "Need more budget";
}

const result = await wallet.runAgent({
  messages: [{ role: 'user', content: 'Research topic...' }],
  config: { model: 'gpt-4o', maxIterations: 15 }
});
```

## Spawn Child Agent

```javascript
const child = await wallet.spawnChildInvite({
  budgetUsd: 10.00,
  name: 'Research Agent'
});

console.log(`Child invite: ${child.inviteCode}`);
// Share with child process
```

## Deploy MCP Server

```javascript
const instance = await wallet.deployJitMcp({
  glamaId: 'notion-mcp',
  glamaNamespace: 'notion',
  glamaSlug: 'notion-mcp',
  toolName: 'My Notion',
  envVars: { NOTION_API_KEY: 'secret_...' }
});

console.log(`Endpoint: ${instance.instance.endpointUrl}`);
```

## Error Handling

```javascript
try {
  const result = await wallet.complete('...');
} catch (error) {
  switch (error.code) {
    case 'BUDGET_EXHAUSTED':
      return "Out of budget";
    case 'SESSION_EXPIRED':
      return "Need new invite";
    default:
      throw error;
  }
}
```

## Full Research Workflow

```javascript
async function research(wallet, topic) {
  // Check budget
  const stats = await wallet.getSpendingStats();
  if (stats.availableUsd < 5.00) {
    return { error: 'Need $5 minimum' };
  }

  // Quick overview
  const overview = await wallet.complete(`Brief overview of ${topic}`);

  // Deep research with tools
  const research = await wallet.runAgent({
    messages: [{ role: 'user', content: `Research ${topic} in depth` }],
    config: { model: 'gpt-4o', maxIterations: 15 }
  });

  return {
    overview: overview.text,
    research: research.response,
    cost: overview.costUsd + research.cost.totalUsd
  };
}
```

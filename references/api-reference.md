# MixrPay SDK Reference

**Package**: `@mixrpay/agent-sdk`

```bash
npm install @mixrpay/agent-sdk
```

## Static Methods

### `AgentWallet.claimInvite(options)`

Claim an invite to get a session key.

```typescript
const result = await AgentWallet.claimInvite({
  inviteCode: string,
  privateKey: string  // Your wallet's private key
});
// Returns: { sessionKey, walletAddress, budgetUsd, expiresAt }
```

## Constructor

```typescript
const wallet = new AgentWallet({
  sessionKey: string,      // Required
  maxPaymentUsd?: number,  // Optional safety limit
  onPayment?: (p) => void  // Optional callback
});
```

## LLM Methods

### `wallet.complete(prompt, options?)`

Simple LLM completion.

```typescript
const result = await wallet.complete('Your prompt', {
  model?: string,        // Default: 'gpt-4o-mini'
  systemPrompt?: string
});
// Returns: { text, costUsd, tokens, model }
```

### `wallet.runAgent(options)`

Multi-turn agentic loop with tools.

```typescript
const result = await wallet.runAgent({
  messages: [{ role: 'user', content: string }],
  config?: {
    model?: string,
    maxIterations?: number,
    tools?: string[],
    systemPrompt?: string
  },
  stream?: boolean,
  onEvent?: (event) => void
});
// Returns: { response, cost, toolsUsed, iterations }
```

## Budget Methods

### `wallet.getSpendingStats()`

```typescript
const stats = await wallet.getSpendingStats();
// Returns: { totalBudgetUsd, spentUsd, availableUsd, canSpawn, expiresAt }
```

### `wallet.getAvailableBudget()`

```typescript
const budget = await wallet.getAvailableBudget();
// Returns: { availableBudget, maxSpawnBudget, canSpawn }
```

## Child Agent Methods

### `wallet.spawnChildInvite(options)`

```typescript
const child = await wallet.spawnChildInvite({
  budgetUsd: number,
  name?: string,
  allowedMerchants?: string[],
  expiresInDays?: number
});
// Returns: { inviteCode, budgetUsd, expiresAt }
```

### `wallet.getChildSessions()`

```typescript
const children = await wallet.getChildSessions();
// Returns: [{ id, name, budgetUsd, spentUsd, status }]
```

## MCP Server Methods

### `wallet.deployJitMcp(options)`

```typescript
const result = await wallet.deployJitMcp({
  glamaId: string,
  glamaNamespace: string,
  glamaSlug: string,
  toolName: string,
  envVars?: Record<string, string>,
  ttlHours?: number  // Default: 24
});
// Returns: { instance: { id, endpointUrl, expiresAt }, payment }
```

### `wallet.listJitInstances(options?)`

```typescript
const instances = await wallet.listJitInstances({
  status?: 'active' | 'stopped' | 'expired'
});
// Returns: [{ id, endpointUrl, toolName, status, expiresAt }]
```

### `wallet.stopJitInstance(id)`

```typescript
await wallet.stopJitInstance('instance_id');
```

## Models

| Model | Best For |
|-------|----------|
| `gpt-4o-mini` | Fast, cheap (default) |
| `gpt-4o` | Complex reasoning |
| `claude-sonnet-4-5` | Long context |
| `claude-haiku-4-5` | Fast |

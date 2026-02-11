# MixrPay Agent SDK Reference

**Package**: `@mixrpay/agent-sdk` v0.8.8

```bash
npm install @mixrpay/agent-sdk
```

---

## Static Methods

### `AgentWallet.claimInvite(options)`

Claim an invite code to get a session key. Used for manual setup when not auto-provisioned.

```typescript
const result = await AgentWallet.claimInvite({
  inviteCode: string,       // e.g., "mixr-abc123..."
  privateKey: `0x${string}`, // Agent's private key (used for signing, NOT transmitted)
  baseUrl?: string           // Default: "https://www.mixrpay.com"
});
```

**Returns** `AgentClaimInviteResult`:
```typescript
{
  sessionKey: string;       // "sk_live_{hex}" — STORE SECURELY
  address: string;          // Derived public address
  sessionKeyId: string;     // Database ID
  expiresAt: Date;
  limits: {
    budgetUsd: number;      // Total budget in USD
    budgetPeriod: string;   // 'daily' | 'monthly' | 'total'
    maxPerTxUsd: number | null;
  };
  allowedMerchants: string[];
  inviterName: string;
}
```

---

## Constructor

```typescript
import { AgentWallet } from '@mixrpay/agent-sdk';

const wallet = new AgentWallet({
  sessionKey: string,          // Required: "sk_live_{hex}" or from env
  maxPaymentUsd?: number,      // Optional per-payment safety limit
  onPayment?: (payment) => void // Optional callback on each payment
});
```

---

## LLM Methods

### `wallet.complete(prompt, options?)`

Simple one-shot LLM completion. Internally uses `runAgent` with no tools.

```typescript
const result = await wallet.complete('Summarize this article: ...', {
  model?: string,         // Default: 'gpt-4o-mini'
  systemPrompt?: string   // Optional system prompt
});
```

**Returns** `CompleteResult`:
```typescript
{
  text: string;           // LLM response
  costUsd: number;        // Cost in USD
  tokens: {
    prompt: number;
    completion: number;
  };
  model: string;          // Model used
}
```

### `wallet.runAgent(options)`

Multi-turn agentic loop with LLM and tool execution.

```typescript
const result = await wallet.runAgent({
  sessionId?: string,     // Optional (derived from session key auth if omitted)
  messages: [{ role: 'user' | 'assistant' | 'system', content: string }],
  config?: {
    model?: string,         // LLM model
    maxIterations?: number, // Max tool-use loops
    tools?: string[],       // Tool names to enable
    systemPrompt?: string
  },
  stream?: boolean,         // Enable SSE streaming (default: false)
  idempotencyKey?: string,  // Unique key for idempotency
  onEvent?: (event) => void // Streaming event callback
});
```

**Returns** `AgentRunResult`:
```typescript
{
  runId: string;
  status: 'completed' | 'failed';
  response: string;         // Final agent response
  iterations: number;
  toolsUsed: string[];
  cost: {
    llmUsd: number;
    toolsUsd: number;
    totalUsd: number;
  };
  tokens: {
    prompt: number;
    completion: number;
  };
}
```

---

## Budget Methods

### `wallet.getSpendingStats()`

Get spending statistics for the current session.

```typescript
const stats = await wallet.getSpendingStats();
```

**Returns** `SpendingStats`:
```typescript
{
  totalSpentUsd: number;
  txCount: number;
  remainingDailyUsd: number | null;   // null = no daily limit
  remainingTotalUsd: number | null;   // null = no total limit
  expiresAt: Date | null;
}
```

### `wallet.getAvailableBudget()`

Get detailed budget breakdown including child allocation info.

```typescript
const budget = await wallet.getAvailableBudget();
```

**Returns** `AvailableBudget`:
```typescript
{
  totalBudget: number;          // Total budget from session in USD
  spent: number;                // Amount spent in USD
  allocatedToChildren: number;  // Amount allocated to children in USD
  available: number;            // Available for spending or spawning
  maxSpawnBudget: number;       // Max budget for next spawn (20% of available)
  canSpawn: boolean;            // Whether spawning is allowed
}
```

### `wallet.getBalance()`

Get the parent wallet's USDC balance on-chain.

```typescript
const balance = await wallet.getBalance();
// Returns: { usdcBalance: string } (human-readable USD string)
```

### `wallet.getSessionKeyInfo()`

Get details about the current session key.

```typescript
const info = await wallet.getSessionKeyInfo();
// Returns: { address, walletId, expiresAt, maxTotal, maxDaily, maxPerTx, allowedMerchants, ... }
```

---

## Child Agent Methods

### `wallet.spawnChildInvite(options)`

Spawn a child agent by creating a delegated invite. Max 20% of available budget.

```typescript
const child = await wallet.spawnChildInvite({
  budgetUsd: number,            // Required: budget in USD
  name: string,                 // Required: name for the child
  allowedMerchants?: string[],  // Must be subset of parent's list
  expiresInDays?: number        // Capped to parent's expiry
});
```

**Returns** `SpawnChildResult`:
```typescript
{
  inviteCode: string;       // Share with child agent
  inviteId: string;         // Database ID
  budgetUsd: number;        // Actual budget (may be capped)
  expiresAt: Date;
  depth: number;            // 1 = child, 2 = grandchild, etc.
  maxSpawnBudget: number;   // How much this child can spawn
  allowedMerchants: string[];
}
```

### `wallet.getChildSessions()`

List all child sessions spawned by this session.

```typescript
const children = await wallet.getChildSessions();
```

**Returns** `ChildSession[]`:
```typescript
{
  inviteId: string;
  inviteCode: string;
  name: string;
  budgetUsd: number;
  spentUsd: number;
  allocatedToChildrenUsd: number;
  availableUsd: number;
  status: 'active' | 'claimed' | 'revoked' | 'expired';
  claimedBy?: string;       // Wallet address
  depth: number;
  canSpawn: boolean;
  expiresAt: string;
  children?: ChildSession[]; // Grandchildren
}
```

---

## MCP Server Methods

### `wallet.searchGlamaDirectory(query)`

Search the Glama MCP server directory.

```typescript
const results = await wallet.searchGlamaDirectory('notion');
// Returns: { servers: GlamaServer[], pageInfo, query }
```

### `wallet.deployJitMcp(options)`

Deploy a Just-In-Time MCP server. Costs $1.00 per deployment.

```typescript
const result = await wallet.deployJitMcp({
  glamaId: string,                  // Glama server ID
  glamaNamespace: string,           // Glama namespace
  glamaSlug: string,                // Glama slug
  toolName: string,                 // Human-readable name
  toolDescription?: string,         // Optional description
  envVars?: Record<string, string>, // API keys, config
  ttlHours?: number                 // Default: 24, max: 168
});
```

**Returns** `DeployJitMcpResult`:
```typescript
{
  instance: {
    id: string;
    endpointUrl: string;    // Private URL with secret token
    toolName: string;
    glamaId: string;
    glamaNamespace: string;
    glamaSlug: string;
    status: 'provisioning' | 'active' | 'stopped' | 'expired' | 'failed';
    mode: 'private' | 'public';
    ttlHours: number;
    expiresAt: Date;
    createdAt: Date;
  };
  payment: {
    method: string;
    amountUsd: number;
    txHash: string;
  };
}
```

### `wallet.listJitInstances(options?)`

List deployed JIT MCP server instances.

```typescript
const instances = await wallet.listJitInstances({
  status?: 'provisioning' | 'active' | 'stopped' | 'expired'
});
// Returns: JitInstance[] (same shape as instance above)
```

### `wallet.stopJitInstance(instanceId)`

Stop a running JIT MCP server. No refund — billed at deploy time.

```typescript
await wallet.stopJitInstance('instance_id');
```

---

## Utility Methods

### `wallet.fetch(url, options?)`

HTTP fetch with automatic x402 payment negotiation.

```typescript
const response = await wallet.fetch('https://api.example.com/paid-resource', {
  method?: string,
  headers?: Record<string, string>,
  body?: string
});
```

### `wallet.runDiagnostics()`

Run a full health check on the wallet, session key, and connectivity.

```typescript
const diag = await wallet.runDiagnostics();
// Returns: { status, checks, issues, recommendations }
```

---

## Error Handling

All methods throw `MixrPayError` with descriptive messages:

```typescript
try {
  const result = await wallet.complete('...');
} catch (error) {
  if (error.code === 'BUDGET_EXHAUSTED') {
    // Budget used up — ask human for more
  }
  if (error.code === 'SESSION_EXPIRED') {
    // Session key expired — need new invite
  }
  if (error.code === 'INSUFFICIENT_BALANCE') {
    // Parent wallet doesn't have enough USDC
  }
}
```

---

## Available Models

| Model | Best For | Approx. Cost |
|-------|----------|-------------|
| `gpt-4o-mini` | Fast, cheap (default) | ~$0.15/1M input |
| `gpt-4o` | Complex reasoning | ~$2.50/1M input |
| `claude-sonnet-4-5` | Long context, analysis | ~$3.00/1M input |
| `claude-haiku-4-5` | Fast, light tasks | ~$0.80/1M input |

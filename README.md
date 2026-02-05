# MixrPay ClawHub Skill

Budget-controlled payments for AI agents. Make LLM calls, deploy MCP servers, and spawn child agents.

## Installation

```bash
npm install @mixrpay/agent-sdk
```

## Quick Start

```javascript
import { AgentWallet } from '@mixrpay/agent-sdk';

// Initialize with your session key
const wallet = new AgentWallet({
  sessionKey: process.env.MIXRPAY_SESSION_KEY
});

// Make LLM calls
const result = await wallet.complete('Summarize this article...');
console.log(result.text);
```

## Features

- **Simple Completions** - `wallet.complete()` for basic LLM calls
- **Agent Runs** - `wallet.runAgent()` for multi-turn with tools
- **Budget Management** - Track spending with `wallet.getSpendingStats()`
- **Nested Spawning** - Create child agents with `wallet.spawnChildInvite()`
- **MCP Servers** - Deploy your own with `wallet.deployJitMcp()`

## Documentation

- [SKILL.md](./SKILL.md) - Full skill documentation
- [API Reference](./references/api-reference.md) - SDK method signatures
- [Examples](./references/examples.md) - Code samples
- [Troubleshooting](./references/troubleshooting.md) - Common issues

## Links

- **Dashboard**: https://mixrpay.com/wallet
- **SDK Docs**: https://mixrpay.com/docs
- **npm**: [@mixrpay/agent-sdk](https://npmjs.com/package/@mixrpay/agent-sdk)

## License

MIT

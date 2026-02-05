# MixrPay Troubleshooting

Common issues and solutions.

## Authentication

### "Invalid session key"
- Verify key starts with `sk_live_` or `sk_test_`
- Check key hasn't been revoked at mixrpay.com/wallet

### "Session expired"
- Request new invite from human
- Check expiration with `wallet.getSpendingStats()`

## Budget Issues

### "Budget exhausted"
- Stop operations, notify human
- Request new invite with more budget

### "Spawn budget exceeded"
- Child budget can't exceed 20% of your available
- Use `getAvailableBudget()` to check max spawn amount

## Server Issues

### "Missing required environment variables"
- Check what API keys the MCP server needs
- Pass them in `envVars` when deploying

### "Tool unavailable"
- Retry after 30 seconds
- Use alternative tool

## Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| `BUDGET_EXHAUSTED` | No funds | Get new invite |
| `SESSION_EXPIRED` | Key expired | Get new invite |
| `SPAWN_BUDGET_EXCEEDED` | >20% requested | Reduce amount |
| `TOOL_UNAVAILABLE` | Service down | Retry |

## Getting Help

- Dashboard: https://mixrpay.com/wallet
- Docs: https://mixrpay.com/docs

# CLAUDE.md - Project Context for AI Assistants

## Project Overview

This is **House Protocol's fork** of the [InverterNetwork floors-indexer](https://github.com/InverterNetwork/floors-indexer). It's an **Envio-based blockchain event indexer** that tracks Floor Markets DeFi protocol events and exposes them via GraphQL.

## Architecture

```
Anvil Node (Railway)     →    Envio Indexer (Hosted)    →    GraphQL API
  - Avalanche fork              - Processes events            - Query indexed data
  - Chain ID: 31337             - Stores in PostgreSQL        - Used by SDK/frontend
```

## Key Configuration

### Contract Addresses (update when redeploying)
```yaml
# config.yaml - networks section
FloorFactory: 0x9d37C149580CBc32BB452ECe7108638120cFaBAd
ModuleFactory: 0x9B0Aa230314e210eD40a725cA48EDeCBEB4891a6
```

### RPC Endpoint
```
https://foundry-production-f209.up.railway.app
```

### Multicall3 (Custom Address)
Deployed at `0x5FbDB2315678afecb367f032d93F642f64180aa3` (NOT canonical).
Configured in `src/rpc-client.ts` - viem uses this for batched RPC calls.

### Envio Hosted
- **Dashboard**: https://envio.dev
- **GraphQL Endpoint**: Check Envio dashboard for current URL (changes per deployment)
- **Plan**: Development (Free) - no env vars, config must be committed
- **Release Branch**: `main` - pushes to main trigger auto-deploy

## Git Workflow

```bash
# Origin (our fork)
origin    https://github.com/House-Protocol/floors-indexer.git

# Upstream (InverterNetwork original)
upstream  https://github.com/InverterNetwork/floors-indexer

# Sync with upstream
git fetch upstream
git merge upstream/main
git push origin main
```

## Key Files

| File | Purpose |
|------|---------|
| `config.yaml` | Envio config - contracts, events, RPC URL, start_block |
| `schema.graphql` | GraphQL schema - all entity definitions |
| `src/rpc-client.ts` | Viem client setup - **Multicall3 address configured here** |
| `src/helpers/token.ts` | Token metadata fetching via Effect API |
| `src/*-handlers.ts` | Event handlers by domain |

## Common Tasks

### Update Contract Addresses
1. Edit `config.yaml` - update addresses in `networks.contracts` section
2. Update `start_block` to block before deployment
3. Commit and push to main
4. Delete old Envio deployment, it will auto-create new one

### Re-index From Scratch
1. Delete deployment in Envio dashboard
2. Push any commit to main (even empty): `git commit --allow-empty -m "Trigger re-index" && git push`
3. New deployment starts fresh

### Check Token Metadata Issues
If tokens show as "Unknown Token" / "UNK":
1. Verify Multicall3 is deployed and address matches `src/rpc-client.ts`
2. Test RPC: `cast call <TOKEN_ADDR> "name()(string)" --rpc-url <RPC_URL>`
3. Delete and re-deploy indexer to clear cached bad data

## Local Development

Requires Docker for local PostgreSQL:
```bash
pnpm install
pnpm codegen
pnpm dev
```

Without Docker, use Envio hosted instead.

## Useful Commands

```bash
# Generate types from schema
pnpm codegen

# Check current block on Anvil
cast block-number --rpc-url https://foundry-production-f209.up.railway.app

# Test token metadata
cast call <TOKEN_ADDR> "name()(string)" --rpc-url https://foundry-production-f209.up.railway.app

# Test Multicall3 exists
cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 "getChainId()(uint256)" --rpc-url https://foundry-production-f209.up.railway.app
```

## GraphQL Query Examples

```graphql
# Get all markets
{ Market { id currentPriceFormatted status } }

# Get tokens
{ Token { id name symbol decimals } }

# Get module registry
{ ModuleRegistry { id floor creditFacility feeTreasury authorizer } }

# Get global registry
{ GlobalRegistry { floorFactoryAddress moduleFactoryAddress } }
```

## Troubleshooting

### "Unknown Token" / "UNK" for token names
- **Cause**: Multicall3 not deployed or wrong address
- **Fix**: Ensure Multicall3 is deployed and address in `src/rpc-client.ts` matches

### Indexer not finding events
- **Cause**: `start_block` is after contract deployment
- **Fix**: Set `start_block` in config.yaml to block before deployment

### Changes not reflecting in Envio
- **Cause**: Envio caches data between deployments
- **Fix**: Delete deployment in dashboard, push to trigger fresh deploy

## Contact / Resources

- **Envio Docs**: https://docs.envio.dev
- **Upstream Repo**: https://github.com/InverterNetwork/floors-indexer
- **Viem Docs**: https://viem.sh

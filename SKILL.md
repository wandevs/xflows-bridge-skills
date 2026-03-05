---
name: xflows-bridge
description: "Cross-chain bridge operations using the xflows CLI (Wanchain XFlows). Use when the user wants to: (1) create or manage crypto wallets for cross-chain use, (2) query supported chains, tokens, pairs, bridges, or DEXes, (3) get cross-chain swap quotes and fee estimates, (4) send cross-chain transactions between EVM chains, (5) track cross-chain transaction status and completion. Triggers on: cross-chain, bridge, swap tokens between chains, xflows, Wanchain, WanBridge, QUiX, move/transfer tokens from Ethereum/BSC/Polygon/Arbitrum/Optimism/Avalanche/Wanchain to another chain, bridge ETH/BNB/USDC/USDT."
---

# XFlows Cross-Chain Bridge

Operate the `xflows` CLI to perform cross-chain bridge transactions via the Wanchain XFlows protocol.

## Prerequisites

Ensure `xflows` is installed: `npm install -g xflows`. Verify with `xflows --version`.

## Core Workflow

For any cross-chain transfer, follow this sequence:

1. **Wallet** -- Ensure a wallet exists (`xflows wallet list`), create if needed
2. **Discover** -- Find supported chains, tokens, and pairs for the route
3. **Quote** -- Get estimated output and fees
4. **Send** -- Execute with `--dry-run` first, then for real
5. **Track** -- Monitor status until completion

## Command Quick Reference

See [references/commands.md](references/commands.md) for complete flag details.

### Wallet Management

```bash
xflows wallet create --name <n>                                    # new wallet
xflows wallet create --name <n> --encrypt --password <pw>          # encrypted
xflows wallet create --name <n> --private-key 0x...                # import key
xflows wallet list                                                 # list all
xflows wallet show --name <n> [--password <pw>]                    # show key
xflows wallet balance --name <n> --chain-id <id> [--password <pw>] # balance
xflows wallet delete --name <n> --force                            # delete
```

### Query Routes

```bash
xflows chains [--chain-id <id>] [--quix]          # supported chains
xflows tokens [--chain-id <id>] [--quix]          # supported tokens
xflows pairs --from-chain <id> [--to-chain <id>]  # bridgeable pairs
xflows bridges                                     # available bridges
xflows dexes                                       # available DEXes
xflows rpc                                         # configured RPC endpoints
```

### Quote

```bash
xflows quote \
  --from-chain <id> --to-chain <id> \
  --from-token <addr> --to-token <addr> \
  --from-address <addr> --to-address <addr> \
  --amount <amount> \
  [--bridge wanbridge|quix] [--slippage 0.01] [--dex wanchain|rubic]
```

### Send Transaction

```bash
xflows send \
  --wallet <name> [--password <pw>] \
  --from-chain <id> --to-chain <id> \
  --from-token <addr> --to-token <addr> \
  --to-address <addr> --amount <amount> \
  [--bridge wanbridge|quix] [--slippage 0.01] [--dry-run] [--gas-limit <n>] [--rpc <url>]
```

### Track Status

```bash
xflows status \
  --hash <txHash> \
  --from-chain <id> --to-chain <id> \
  --from-token <addr> --to-token <addr> \
  --from-address <addr> --to-address <addr> \
  --amount <amount> \
  [--poll --interval 10]
```

## Key Concepts

### Token Addresses

- Native tokens (ETH, BNB, MATIC, WAN, etc.): `0x0000000000000000000000000000000000000000`
- ERC-20 tokens: find via `xflows tokens --chain-id <id>`

### Common Chain IDs

| Chain | ID | Chain | ID |
|-------|----|-------|----|
| Ethereum | 1 | Arbitrum | 42161 |
| BSC | 56 | Optimism | 10 |
| Polygon | 137 | Base | 8453 |
| Avalanche | 43114 | Wanchain | 888 |

### Encrypted Wallets

When a wallet was created with `--encrypt`, every command that uses the private key (`send`, `wallet show`, `wallet balance`) requires `--password <pw>`.

### Wanchain Gas Rule

Sending from Wanchain (chainId 888) automatically enforces minimum gasPrice of 1 gwei. No manual action needed.

### Status Codes

| Code | Meaning | Terminal? |
|------|---------|-----------|
| 1 | Success | Yes |
| 2 | Failed | Yes |
| 3 | Processing | No |
| 4/5 | Refunded | Yes |
| 6 | Trusteeship (manual intervention) | Yes |
| 7 | Risk transaction (AML flagged) | Yes |

## Patterns

### Find a Token Address

```bash
xflows tokens --chain-id 1 | jq '.data[] | select(.tokenSymbol == "USDC")'
```

### Safe Execution (always dry-run first)

```bash
# Preview
xflows send --wallet alice --from-chain 1 --to-chain 56 \
  --from-token 0x0000000000000000000000000000000000000000 \
  --to-token 0x0000000000000000000000000000000000000000 \
  --to-address 0xRecipient --amount 0.1 --dry-run

# Execute (remove --dry-run)
xflows send --wallet alice --from-chain 1 --to-chain 56 \
  --from-token 0x0000000000000000000000000000000000000000 \
  --to-token 0x0000000000000000000000000000000000000000 \
  --to-address 0xRecipient --amount 0.1
```

### Poll Until Complete

```bash
xflows status --hash 0xTxHash \
  --from-chain 1 --to-chain 56 \
  --from-token 0x0000000000000000000000000000000000000000 \
  --to-token 0x0000000000000000000000000000000000000000 \
  --from-address 0xSender --to-address 0xReceiver \
  --amount 0.1 --poll --interval 10
```

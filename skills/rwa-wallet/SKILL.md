---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via the
  rwa CLI. Use for wallet creation, import, export/backup, encryption,
  sending/withdrawing funds, or reclaiming rent. Triggers: "create wallet",
  "generate keys", "import seed phrase", "export key", "backup wallet",
  "show address", "send SOL", "send USDC", "withdraw", "transfer",
  "reclaim rent", "encrypt wallet", "passphrase".
---

# RWA Wallet

Wallet setup + transfers for the rwa CLI. Use `--json` for agent flows.

## Hard rules

- Encryption is the **default** for new wallets; `--allow-plaintext` opts out (discouraged).
- `keys generate` prints a **12-word recovery phrase once** (Phantom/Solflare-compatible). Encrypted wallets store it — recover later with `keys export --reveal`; plaintext wallets never store it, so the user MUST save it at creation.
- **USDC alone is enough to trade.** Swaps are usually gasless, and the CLI auto-buys SOL for fees when it runs low (dynamic 5–25 USDC; reported as `gas_refuel` in trade JSON; disable with `RWA_NO_AUTO_GAS=1`). `send`/`reclaim` still need SOL.
- `send` transfers assets to another wallet; it does NOT sell them (use `gm sell` to swap).
- `send USDC all` / `send SOL all` send the **entire** balance. For an exact amount, pass the value from CLI output — never manual arithmetic.
- GM-token amounts are in **raw tokens**; wallets (Phantom) display raw × dividend multiplier, so "send what the wallet shows" can overshoot — the CLI then fails fast with `insufficient_funds` naming both numbers. Use `all` to avoid it.
- `send SOL all` auto-reserves the tx fee; transfers use full on-chain precision.
- After liquidation, run `reclaim` before the final `send SOL all`.
- Never print secrets unless the user explicitly asks; `keys export` requires `--reveal` in `--json` mode.

## Intent → command

| Intent | Command |
|---|---|
| Install CLI | `curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh \| sh` |
| Update CLI to latest | `rwa update -y` (`--check` to preview; verifies SHA-256, fail-closed) |
| Show address | `rwa keys show` |
| New wallet | `rwa keys generate` (encrypted; prints recovery phrase once) |
| Import wallet | `rwa keys import --seed-phrase "..."` (or `--private-key <B58>` / `--file <PATH>`) |
| Export / back up | `rwa --json keys export --reveal` → base58 key (Phantom format), JSON array, mnemonic |
| Encrypt / decrypt | `rwa keys encrypt` / `rwa keys decrypt` |
| Multiple wallets | `rwa keys add <name> --path <p>` · `keys list` · `keys use <name>` · `rwa --wallet <name> gm ...` |
| Preview transfer | `rwa --json gm send <TOKEN> <AMT> <ADDR> --dry-run` |
| Send USDC / SOL / token | `rwa --json gm send USDC 100 <ADDR> -y` |
| Withdraw everything | `send USDC all` then `send SOL all` |
| Reclaim empty accounts | `rwa --json gm reclaim` (or `--token <SYM>`) |

## Passphrase

If the wallet is encrypted, commands prompt for the passphrase. For scripts:

```bash
export RWA_PASSPHRASE="my passphrase"
```

## Canonical flows

```bash
# Fund-and-trade (USDC only — no SOL needed):
rwa keys generate && rwa keys show        # create, get the address, fund USDC
rwa --json gm buy TSLA 10 -y              # CLI auto-buys gas SOL if needed

# Full withdrawal:
rwa --json gm close-all -y                # 1. liquidate positions (parallel)
rwa --json gm reclaim                     # 2. recover SOL rent from empty accounts
rwa --json gm send USDC all <ADDR> -y     # 3. withdraw USDC
rwa --json gm send SOL  all <ADDR> -y     # 4. withdraw remaining SOL
```

## Errors → action

| Error | Agent action |
|---|---|
| `No wallet found` | Run `rwa keys generate` |
| `insufficient_funds` (SOL, on send/reclaim) | Fund SOL, or free some by selling; trades usually don't need SOL |
| `keys export` refused | Add `--reveal` (mandatory with `--json`); warn the user it prints secrets |
| RPC `unavailable` (exit 75) | Wait a few seconds; on repeats set `RWA_RPC_URL` to a dedicated endpoint |
| `rwa update` → `network`/`rate_limited` (exit 75) | Transient — retry shortly |

## Notes

- The install script prefers a release binary, falls back to source build.
- For faster/private RPC: `export RWA_RPC_URL=https://your-endpoint`.

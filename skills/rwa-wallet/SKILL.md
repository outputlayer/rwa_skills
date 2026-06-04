---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via the
  rwa CLI. Use for wallet creation, import, encryption, sending/withdrawing
  funds, or reclaiming rent. Triggers: "create wallet", "generate keys",
  "import seed phrase", "show address", "send SOL", "send USDC", "withdraw",
  "transfer", "reclaim rent", "encrypt wallet", "passphrase".
---

# RWA Wallet

Wallet setup + transfers for the rwa CLI. Use `--json` for agent flows.

## Hard rules

- Prefer **encrypted** wallets for new setups (`--encrypt`).
- `send` transfers assets to another wallet; it does NOT sell them (use `gm sell` to swap).
- `send USDC all` / `send SOL all` send the **entire** balance, not just recent proceeds. For an exact amount, pass the value from CLI output — never manual arithmetic.
- `send SOL all` auto-reserves the tx fee; transfers use full on-chain precision.
- After liquidation, run `reclaim` before the final `send SOL all`.
- No `--dry-run` for `keys`; use `keys show` for low-risk inspection. Never mix `-y` with `--dry-run`.

## Intent → command

| Intent | Command |
|---|---|
| Install CLI | `curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh \| sh` |
| Update CLI to latest | `rwa update -y` (`--check` to preview; verifies SHA-256, fail-closed) |
| Show address | `rwa keys show` |
| New wallet | `rwa keys generate --encrypt` |
| Import wallet | `rwa keys import --seed-phrase "..." --encrypt` (or `--private-key <B58>` / `--file <PATH>`) |
| Encrypt / decrypt | `rwa keys encrypt` / `rwa keys decrypt` |
| Preview transfer | `rwa --json gm send <TOKEN> <AMT> <ADDR> --dry-run` |
| Send USDC / SOL / token | `rwa --json gm send USDC 100 <ADDR> -y` |
| Withdraw everything | `send USDC all` then `send SOL all` |
| Reclaim empty accounts | `rwa --json gm reclaim` (or `--token <SYM>`) |

## Passphrase

If `~/.config/rwa/key.age` exists, commands prompt for the passphrase. For scripts:

```bash
export RWA_PASSPHRASE="my passphrase"
```

## Canonical withdrawal flow

```bash
rwa --json gm close-all -y                  # 1. liquidate positions
rwa --json gm reclaim                       # 2. recover SOL rent from empty accounts
rwa --json gm send USDC all <ADDR> -y       # 3. withdraw USDC
rwa --json gm send SOL  all <ADDR> -y       # 4. withdraw remaining SOL
```

## Errors → action

| Error | Agent action |
|---|---|
| `No wallet found` | Run `rwa keys generate --encrypt` |
| `Insufficient SOL` | Fund wallet with SOL (CLI reserves fees; no auto-convert from USDC) |
| RPC `unavailable` / `all RPC endpoints failed` | Wait a few seconds; on repeats set `RWA_RPC_URL` to a dedicated endpoint |
| `rwa update` → `rate_limited` / `checksum_mismatch` | Retry shortly; on repeat reinstall via the install.sh one-liner |

## Notes

- The install script prefers a release binary, falls back to source build.
- For faster/private RPC: `export RWA_RPC_URL=https://your-endpoint`.

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
| New wallet | `rwa keys generate` (encrypted; prints recovery phrase once). `--json` → `{status,pubkey,path,encrypted,mnemonic}` — CAPTURE `mnemonic` |
| Import wallet | `rwa keys import --seed-phrase "..."` (or `--private-key <B58>` / `--file <PATH>`). Non-default account: add `--account <N>` (→ `m/44'/501'/N'/0'`, Phantom "Account N+1") or `--derivation-path "<PATH>"` |
| Export / back up | `rwa --json keys export --reveal` → base58 key (Phantom format), JSON array, mnemonic |
| Encrypt / decrypt | `rwa keys encrypt` / `rwa keys decrypt` (both speak `--json`). ADMIN-CLASS since 0.7.9: `decrypt` and `export --reveal` take the passphrase ONLY from a live TTY — `RWA_PASSPHRASE`/keychain are NOT consulted; headless → `error_kind: interactive_required` (exit 1) |
| Store passphrase for headless trading | `rwa keys store-passphrase [--wallet N]` puts it in the OS keychain (verified first). Then trading/reads run without `RWA_PASSPHRASE`. `keys forget-passphrase` removes it. `RWA_KEYRING_DISABLE=1` skips the keychain |
| Restrict `send` recipients | `rwa keys policy show / allow <ADDR> [--label L] / remove <ADDR>` — a per-wallet whitelist stored inside the encrypted payload; `allow`/`remove` need a typed passphrase (admin-class). Encrypted wallets only |
| Multiple wallets | `rwa keys add <name> --path <p>` · `keys list` · `keys use <name>` · `rwa --wallet <name> gm ...` |
| Preview transfer | `rwa --json gm send <TOKEN> <AMT> <ADDR> --dry-run` |
| Send USDC / SOL / token | `rwa --json gm send USDC 100 <ADDR> -y` |
| Withdraw everything | `send USDC all` then `send SOL all` |
| Reclaim empty accounts | `rwa --json gm reclaim` (or `--token <SYM>`) |

## Passphrase (two classes since 0.7.9)

- **Operational** (trading, `keys show`, `keys policy show`, reads): passphrase resolves `RWA_PASSPHRASE` → OS keychain → interactive prompt. Recommended desktop-agent setup: encrypted wallet + `keys store-passphrase` (keychain), NO env passphrase — the agent trades headless but never learns the secret.
- **Admin** (`keys decrypt`, `keys export --reveal`, `keys policy allow/remove`, `keys store-passphrase`): live TTY prompt ONLY — env/keychain deliberately ignored; headless → `interactive_required`. So an agent can't strip encryption, export the key, or edit the send-whitelist.

```bash
export RWA_PASSPHRASE="my passphrase"   # operational commands only (servers/CI)
```

Caveat: with `RWA_PASSPHRASE` set in the env, a co-resident process can read it, collapsing the admin-class guarantee (and the send-policy escape-hatch second factor) to env-leak level. Keychain-only, no-env setup avoids this. Keychain isolation is macOS-strong; on Linux/Windows any same-user process can read the stored item.

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
| `interactive_required` (exit 1) | Admin-class command (`decrypt`/`export --reveal`/`policy allow`/`store-passphrase`) run headless — the human must type the passphrase at a real terminal; env/keychain won't unlock it. Don't retry as-is |
| `recipient_not_allowed` (exit 1, on `send`) | Recipient isn't in the wallet's send-policy whitelist; a HUMAN adds it via `rwa keys policy allow <ADDR>`. Do NOT attempt to bypass — stop and tell the user |
| RPC `unavailable` (exit 75) | Wait a few seconds; on repeats set `RWA_RPC_URL` to a dedicated endpoint |
| `rwa update` → `network`/`rate_limited` (exit 75) | Transient — retry shortly |

## Notes

- The install script prefers a release binary, falls back to source build.
- For faster/private RPC: `export RWA_RPC_URL=https://your-endpoint`.

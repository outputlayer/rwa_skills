---
name: rwa-wallet
description: >
  Set up and manage Solana wallets for trading tokenized stocks via the rwa CLI.
  Use when user asks about wallet setup, key generation, importing seed phrase,
  showing wallet address, or installing the rwa CLI. Triggers: "create wallet",
  "generate keys", "import seed phrase", "show address", "install rwa",
  "fund wallet", "setup trading".
---

# RWA Wallet

Create and manage Solana wallets for trading tokenized stocks.

## Install CLI

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

Or from source (requires Rust 1.91+):
```bash
cargo install --git https://github.com/outputlayer/rwa_cli --bin rwa
```

Verify: `rwa --version`

If `rwa` not found after install, add to PATH:
```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

## Wallet Commands

### Generate New Wallet

```bash
rwa keys generate
```

Creates a new Solana keypair at `~/.config/rwa/id.json`. Outputs the public address and a 12-word seed phrase. **Save the seed phrase** — it cannot be recovered.

### Import Existing Wallet

```bash
rwa keys import --seed-phrase "word1 word2 ... word12"
```

### Show Wallet Info

```bash
rwa keys show
```

Outputs: public address + key file path.

## Funding

Before trading, fund the wallet with:
- **SOL** — for transaction fees (~0.01 SOL per swap)
- **USDC** — for buying tokens (mint: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`)

Send SOL and USDC to the wallet address shown by `rwa keys show`.

## RPC Configuration

Default: public Solana RPC (rate-limited). For better reliability:

```bash
# Environment variable (persistent)
export RWA_RPC_URL=https://your-rpc-provider.com

# Or per-command flag
rwa --rpc-url https://your-rpc-provider.com --json gm portfolio
```

Free RPC providers: [Helius](https://helius.dev/), [QuickNode](https://quicknode.com/), [Alchemy](https://alchemy.com/)

## Security

- Wallet key stored at `~/.config/rwa/id.json` with `0600` permissions (Unix)
- Never output private keys or seed phrases
- Keep seed phrase backed up offline

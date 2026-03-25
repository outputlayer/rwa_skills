# Install and Recovery

## CLI Not Found

If `rwa` is not found, install it:

```bash
# Via install script (installs Rust if missing):
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash

# Or from source (requires Rust 1.91+):
cargo install --git https://github.com/outputlayer/rwa_cli --bin rwa
```

After install, verify:
```bash
rwa --version
```

If `rwa` is still not found after install, the binary is at `~/.cargo/bin/rwa`. Add to PATH:
```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

## Wallet Setup

A wallet is required for trading (buy/sell) and viewing your own portfolio.

```bash
# Generate new wallet
rwa keys generate

# Or import existing wallet
rwa keys import --seed-phrase "word1 word2 ... word12"

# Check wallet address
rwa keys show
```

Wallet is stored at `~/.config/rwa/id.json`. Fund it with:
- **SOL** — for transaction fees (~0.01 SOL per swap)
- **USDC** — for buying tokens

## RPC Errors

The CLI uses 3 public Solana RPC endpoints with automatic rotation. If all fail:

**Error:** `Solana RPC unavailable (all endpoints rate-limited or down)`

**Solutions:**
1. Wait a few seconds and retry — rate limits are temporary
2. Set a private RPC endpoint:
   ```bash
   export RWA_RPC_URL=https://your-rpc-provider.com
   rwa --json gm portfolio
   ```
   Or per-command:
   ```bash
   rwa --rpc-url https://your-rpc-provider.com --json gm portfolio
   ```

**Free RPC providers:**
- [Helius](https://helius.dev/) — free tier with generous limits
- [QuickNode](https://quicknode.com/) — free Solana endpoint
- [Alchemy](https://alchemy.com/) — free Solana tier

## Market Hours

Ondo GM trading is available 24/5: Sunday 8:00 PM ET — Friday 8:00 PM ET.

If a trade fails with "market closed", check hours:
```bash
rwa --json gm hours
```

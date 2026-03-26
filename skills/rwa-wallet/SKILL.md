---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via rwa CLI.
  Triggers: "create wallet", "generate keys", "import seed phrase", "show address",
  "install rwa", "fund wallet", "send SOL", "send USDC", "withdraw", "transfer",
  "reclaim rent", "close accounts".
---

# Install

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

Or: `cargo install --git https://github.com/outputlayer/rwa_cli --bin rwa`

# Wallet

```bash
rwa keys generate                                    # Create new keypair (~/.config/rwa/key.json)
rwa keys import --seed-phrase "word1 word2 ... word12"  # Import from mnemonic
rwa keys import --private-key <BASE58>               # Import from private key
rwa keys import --file <PATH>                        # Import from key file
rwa keys show                                        # Show address + key path
```

Fund wallet with SOL (for fees) + USDC (for trading) at the address from `rwa keys show`.

# Send / Withdraw

```bash
rwa --json gm send SOL 1.5 <ADDR> -y     # Send exact SOL
rwa --json gm send SOL all <ADDR> -y     # Send max SOL (auto-reserves tx fee from RPC)
rwa --json gm send USDC 100 <ADDR> -y    # Send exact USDC
rwa --json gm send USDC all <ADDR> -y    # Send full USDC balance (raw precision)
rwa --json gm send TSLA 0.5 <ADDR> -y   # Send GM tokens (Token-2022)
```

After selling: use exact USDC amount from sell result, not `send USDC all`.

# Reclaim rent

```bash
rwa --json gm reclaim                    # Close all empty token accounts
rwa --json gm reclaim --token TSLA       # Close only for specific token
```

# RPC

Default: public Solana RPC. For reliability: `export RWA_RPC_URL=https://your-rpc.com`

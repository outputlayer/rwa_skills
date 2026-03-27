---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via rwa CLI.
  Triggers: "create wallet", "generate keys", "import seed phrase", "show address",
  "install rwa", "send SOL", "send USDC", "withdraw", "transfer", "reclaim rent".
---

# Install

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

# Wallet

```bash
rwa keys generate                                      # Create keypair (~/.config/rwa/id.json)
rwa keys import --seed-phrase "word1 word2 ... word12"  # Import mnemonic
rwa keys import --private-key <BASE58>                  # Import private key
rwa keys show                                           # Show address + key path
```

Fund with USDC (trading) at address from `rwa keys show`. SOL optional — Jupiter pays gas for swaps.

# Send / Withdraw

```bash
rwa --json gm send SOL 1.5 <ADDR> -y     # Send exact SOL
rwa --json gm send SOL all <ADDR> -y     # Drain SOL (auto-reserves tx fee)
rwa --json gm send USDC 100 <ADDR> -y    # Send exact USDC
rwa --json gm send USDC all <ADDR> -y    # Drain USDC (full precision)
rwa --json gm send TSLA 0.5 <ADDR> -y   # Send GM tokens
```

After selling positions: use exact USDC amount from sell result for precision.

# Reclaim rent

```bash
rwa --json gm reclaim                    # Close all empty token accounts
rwa --json gm reclaim --token TSLA       # Close only specific token
```

# RPC

Default: public Solana RPC. For speed: `export RWA_RPC_URL=https://your-rpc.com`

---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via rwa CLI.
  Triggers: "create wallet", "generate keys", "import seed phrase", "show address",
  "install rwa", "send SOL", "send USDC", "withdraw", "transfer", "reclaim rent",
  "encrypt wallet", "passphrase".
---

# Install

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

# Wallet — plaintext

```bash
rwa keys generate                                       # Create keypair (~/.config/rwa/key.json)
rwa keys import --seed-phrase "word1 word2 ... word12"  # Import mnemonic
rwa keys import --private-key <BASE58>                  # Import private key
rwa keys import --file <PATH>                           # Import from key file
rwa keys show                                           # Show address + key path
```

Fund with USDC (trading) at address from `rwa keys show`. SOL optional — Jupiter pays gas for swaps.

# Wallet — encrypted (age)

```bash
rwa keys generate --encrypt                             # Create keypair encrypted with passphrase
rwa keys import --seed-phrase "..." --encrypt           # Import + encrypt immediately
rwa keys encrypt                                        # Convert existing key.json → key.age
rwa keys decrypt                                        # Convert key.age → key.json
```

When `~/.config/rwa/key.age` exists, every command prompts for passphrase automatically.
Skip the prompt by setting the env var:

```bash
export RWA_PASSPHRASE="my passphrase"
rwa --json gm portfolio
```

# Send / Withdraw

```bash
rwa --json gm send SOL 1.5 <ADDR> -y      # Send exact SOL
rwa --json gm send SOL all <ADDR> -y      # Drain SOL (auto-reserves tx fee)
rwa --json gm send USDC 100 <ADDR> -y     # Send exact USDC
rwa --json gm send USDC all <ADDR> -y     # Drain USDC (full precision)
rwa --json gm send TSLA 0.5 <ADDR> -y    # Send GM tokens
rwa --json gm send USDC 100 <ADDR> --dry-run  # Preview without sending
```

**After selling positions**: use the EXACT USDC amount from sell/close-all result — avoids float precision issues.

# Reclaim rent

Empty token accounts lock ~0.002 SOL each. Reclaim after selling:

```bash
rwa --json gm reclaim                     # Close all empty token accounts
rwa --json gm reclaim --token TSLA        # Close only one specific token
```

# RPC

Default: public Solana mainnet-beta. For faster/private RPC:

```bash
export RWA_RPC_URL=https://your-rpc-endpoint.com
```

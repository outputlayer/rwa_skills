# Ondo Global Markets (GM) Tokens

## Overview

Ondo Global Markets offers **264 tokenized stocks and ETFs** on Solana. These are total return trackers — not direct equity ownership.

## Key Characteristics

- **Total return trackers**: dividends are reinvested, so 1 token ≠ 1 share
- **Solana Token-2022**: all GM tokens use the Token-2022 program (`TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`)
- **Trading pair**: all tokens trade against USDC (`EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`)
- **Swap routing**: via Jupiter Ultra API on Solana
- **Market hours**: 24/5, Sunday 8:00 PM ET — Friday 8:00 PM ET

## Symbol Formats

Both formats are accepted and resolve to the same token:
- Short: `TSLA`, `AAPL`, `NVDA`, `SPY`
- Full: `TSLAon`, `AAPLon`, `NVDAon`, `SPYon`

## Popular Tokens

| Symbol | Name | Category |
|--------|------|----------|
| TSLA | Tesla | Stock |
| AAPL | Apple | Stock |
| NVDA | Nvidia | Stock |
| AMZN | Amazon | Stock |
| GOOGL | Alphabet | Stock |
| MSFT | Microsoft | Stock |
| SPY | S&P 500 ETF | ETF |
| QQQ | Nasdaq 100 ETF | ETF |
| IWM | Russell 2000 ETF | ETF |
| GLD | Gold ETF | ETF |

Use `rwa --json gm list` for the complete list of all 264 tokens with mint addresses.

## Price Discovery

```bash
# Current price via quote (how many tokens for 100 USDC)
rwa --json gm quote TSLA 100

# Historical prices
rwa --json gm history TSLA -r 1M    # 1 month
rwa --json gm history TSLA -r 1Y    # 1 year
rwa --json gm history TSLA -r ALL   # all time
```

## Trading Mechanics

1. **Buy**: USDC → token (via Jupiter swap)
2. **Sell**: token → USDC (via Jupiter swap)
3. **Slippage**: handled by Jupiter Ultra API
4. **Gas**: ~0.01 SOL per transaction
5. **Fees**: Jupiter swap fees (typically < 0.1%)

## Important Notes

- Tokens are **not** equity. They track the total return of the underlying asset.
- US persons restrictions may apply per Ondo's terms.
- Always check `rwa gm hours` before trading — transactions will fail if market is closed.

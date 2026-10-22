# Walt Perps — follow-up check (2026-10-22 15:55 UTC)

This is a follow-up to [the initial research log](./walt-perps-research-2026-10-22.md). Read-only checks were repeated through the connected Walt account. No trading or account-setting writes were performed.

## Fresh snapshot

- Walt server time: `2026-10-22T15:55:44Z`
- Readiness: `ok=true`; 127 contracts; 3 open positions; no reported readiness issues.
- Available balance: `8.93276700 USDT`
- Latest rates returned in this check:
  - BTC-USD-PERP: `81664.6`
  - ETH-USD-PERP: `2398.94`
  - SOL-USD-PERP: `104.652`
  - HYPE-USD-PERP: `78.0726`
  - ZEC-USD-PERP: `1232.841`

These are point-in-time latest rates, not a historical candle series.

## Open positions and approximate mark-to-entry movement

| Contract | Side | Entry | Latest rate | Base size | Approx. gross PnL from entry* |
|---|---|---:|---:|---:|---:|
| BTC-USDC | Long | 81,986.60 | 81,664.60 | 0.00007 | -$0.0225 |
| ETH-USDC | Short | 2,401.98 | 2,398.94 | 0.0024 | +$0.0073 |
| SOL-USDC | Short | 104.680 | 104.652 | 0.057 | +$0.0016 |

\*Simple base-size × price-difference estimate before opening/closing fees, funding, slippage, and accounting specifics. It is not realized PnL and should not be treated as an exact wallet balance change.

Position settings still reported by Walt:

| Contract | Margin | Leverage | TP trigger | SL trigger | Liquidation |
|---|---:|---:|---:|---:|---:|
| BTC long | 2.869500 USDC | 2× | 82,791.32 | 81,151.88 | 41,491.64 |
| ETH short | 1.921889 USDC | 3× | 2,350.00 | 2,440.00 | 3,164.79 |
| SOL short | 2.979361 USDC | 2× | 102.20 | 106.40 | 153.27 |

## Order audit finding

The guide states that `trigger_price_quote` is the configured trigger and `price_quote` is widened for slippage. The position payload still shows the following pairs:

- BTC long TP trigger 82,791.32 vs TP `price_quote` 80,307.5804; SL trigger 81,151.88 vs SL `price_quote` 78,717.3236.
- ETH short TP trigger 2,350.00 vs TP `price_quote` 2,420.50; SL trigger 2,440.00 vs SL `price_quote` 2,513.20.
- SOL short TP trigger 102.20 vs TP `price_quote` 105.266; SL trigger 106.40 vs SL `price_quote` 109.592.

Those differences are materially larger than a simple 1% widening in some cases. The order listing for contract IDs 0, 1, and 2 returned filled market-entry orders; it did not expose separate pending/processing orders for the TP/SL legs. This is not enough evidence to declare the protection broken, but it is also not enough to certify it as correct. **Manual review in the Walt interface or authoritative venue order detail is required before relying on the displayed TP/SL.** No changes were made because changing protection is a consequential write that needs a separate, exact preview and explicit confirmation.

## Trading decision

**NO NEW POSITION.** Reasons:
1. The account already has three open positions and about 7.77 USDC of reported isolated margin committed, while available funds are shown as 8.93 USDT. Do not assume USDT and USDC collateral are perfectly interchangeable.
2. The checked interface exposes latest rates but no candle-history/ OHLCV endpoint in the tools available in this workflow. Therefore a closed-candle breakout/retest, ATR, and volume confirmation could not be validated.
3. HYPE and ZEC remain watch-only; their point quotes alone do not establish an entry edge.
4. Market-order metadata for HYPE/ZEC lists 1% assumed price slippage and 0.09% taker fee. Actual execution can be worse; a preview is not a guaranteed fill.
5. The TP/SL representation needs clarification before adding risk.

## Evidence labels

- **OBSERVED:** server time, readiness, available USDT balance, position fields, latest rates, and returned order-list contents.
- **DERIVED:** rough gross price movement shown above.
- **NOT_TESTED:** strategy expectancy, candle-based signal, true fill/slippage distribution, and live TP/SL trigger behavior.
- **NEEDS VERIFICATION:** the material gaps between TP/SL trigger values and slippage-adjusted `price_quote` fields.

## Next steps

1. Inspect each position's TP/SL in Walt's UI / authoritative order details.
2. If the UI confirms an inconsistency, prepare a correction preview for only the affected leg(s); show exact values and wait for explicit confirmation before executing.
3. Obtain candle history from a reliable market-data source, run a reproducible backtest with fees/slippage, and only then consider a new market-order preview.

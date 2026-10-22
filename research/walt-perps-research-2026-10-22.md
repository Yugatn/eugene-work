# Walt Perps — research log (2026-10-22)

**Snapshot time:** 2026-10-22 15:49 UTC (Walt server time)  
**Scope:** read-only account review and market screening. No new orders were placed.  
**Status:** HOLD / no new entry until risk controls and a repeatable signal are verified.

> This is a timestamped research note, not a guarantee of returns or a signal to trade. Perpetual futures can lose the entire isolated margin and can incur funding and execution costs.

## 1. Account snapshot

- Walt readiness: `ok=true`, 127 contracts, 3 open positions, no reported readiness blockers.
- Available balance: **8.932767 USDT**.
- Open-position collateral reported by Walt: approximately **7.77075 USDC** across the three positions. USDT and USDC are not assumed to be exactly interchangeable; check the venue's collateral conversion and actual free margin before any order.
- Current prices are point-in-time Walt rate reads. They are not OHLC candles and do not establish a trend by themselves.

| Contract | Side | Leverage | Entry | Walt rate | Size (base) | Isolated margin | Approx. unrealized PnL* |
|---|---:|---:|---:|---:|---:|---:|---:|
| BTC-USDC | Long | 2× | 81,986.60 | 81,592.90 | 0.00007 BTC | 2.869500 USDC | -0.0276 USD |
| ETH-USDC | Short | 3× | 2,401.98 | 2,396.64 | 0.0024 ETH | 1.921889 USDC | +0.0128 USD |
| SOL-USDC | Short | 2× | 104.680 | 104.618 | 0.057 SOL | 2.979361 USDC | +0.0035 USD |

\*Approximation from base size × price change, before fees/funding and any execution effects; not a realized PnL.

### Existing TP/SL settings reported by Walt

| Contract | TP trigger | SL trigger | Liquidation price |
|---|---:|---:|---:|
| BTC long | 82,791.32 | 81,151.88 | 41,491.64 |
| ETH short | 2,350.00 | 2,440.00 | 3,164.79 |
| SOL short | 102.20 | 106.40 | 153.27 |

Walt's position payload also reports separate `price_quote` values for auto-close orders. The Perps guide says `trigger_price_quote` is the configured trigger and `price_quote` is widened for slippage. Those values differ materially, so the actual trigger semantics and execution tolerance should be verified in the venue UI/order details. Do not silently treat the widened `price_quote` as the trigger.

Read-only order listing for contracts 0, 1, 2, 24 and 90 found **no pending/processing orders** at snapshot time. That does not, by itself, prove that the position-level TP/SL will execute as intended; verify the configured auto-close details directly.

## 2. Market screen

| Market | Walt rate at snapshot | Initial decision | Reason |
|---|---:|---|---|
| BTC-PERP | 81,592.90 | Manage existing long; no add | Price below the 81,986.60 entry; avoid adding exposure before checking the existing stop and venue/market divergence. |
| ETH-PERP | 2,396.64 | Manage existing short; no add | Price only modestly below entry; expected edge is too small to justify another trade without candle-based confirmation. |
| SOL-PERP | 104.618 | Manage existing short; no add | Price near entry; do not increase risk for a marginal unrealized gain. |
| HYPE-PERP | 77.8910 | Watch only | Recent public reporting shows volatile movement and inconsistent snapshots across venues. A single rate is not a short/long signal. |
| ZEC-PERP | 1,231.190 | Watch only | Recent reports describe a strong weekly advance and sharp intraday swings. Chasing a move or fading it without candle confirmation has poor evidential support. |

### External context (cross-check, not execution price)

- Coin Edition's 2026-10-22 recap reported BTC at about $82,843, ETH at $2,443, SOL at $105.51, ZEC at $1,294.75 and HYPE at $78.88, with ZEC up 16.1% over seven days. These differ from the Walt snapshot, especially for ZEC. Cross-venue and timestamp differences are material; use Walt's own live rate for execution previews and avoid treating external prices as directly executable.
- CoinMarketCap's 2026-10-21 historical snapshot showed BTC $84,011.97, ETH $2,470.77, SOL $106.79, ZEC $1,311.60 and HYPE $80.58. This is historical context, not a current quote.
- A 2026-10-22 KuCoin news brief described a broad sell-off followed by possible rebound signals for BTC, HYPE and ZEC. It is commentary, not a validated trading rule.

Sources:
- Coin Edition, 2026-10-22 recap: https://coinedition.com/24-hour-crypto-recap-heres-what-happened-in-the-market-81/
- CoinMarketCap, historical snapshot 2026-10-21: https://coinmarketcap.com/historical/20261021/
- KuCoin, 2026-10-22 market brief: https://www.kucoin.com/news/flash/bitcoin-hyperliquid-and-zcash-may-rebound-after-market-sell-off

## 3. Why no new trade was opened

1. The available balance is small relative to current collateral already committed. Preserve a cash buffer rather than adding correlated exposure.
2. Walt's contract metadata lists **1% price slippage** for HYPE and ZEC market orders and a taker fee of **0.09%**. Market orders are not limit orders; the reference price is not a guaranteed fill. With a small account, fees and adverse execution can consume a meaningful fraction of a small target.
3. The latest rate endpoint provides a single latest price, not the multi-candle history needed to verify breakouts, retests, ATR, or trend structure.
4. Public market snapshots and Walt quotes differ. That reinforces the need to use the connected venue's own price and to require a fresh order preview.
5. No pending/processing orders were found in the checked contracts, but the auto-close trigger versus slippage-adjusted price values need inspection before increasing risk.

## 4. Entry criteria for the next review

**Do not enter solely because price moved.** Before considering an entry, collect at least 1h and 15m OHLCV (or equivalent candles), volume, recent swing high/low, funding, spread/slippage estimate and the current position/order state.

A candidate is eligible for a preview only when all are true:
- directional structure is clear on 1h and confirmed on 15m;
- breakout is closed-candle confirmed and preferably retested (no entry on an unconfirmed wick);
- estimated reward/risk after fees and slippage is at least 2:1;
- stop is beyond an invalidation level and risk is sized from the stop distance, not from maximum leverage;
- the position does not duplicate existing correlated exposure;
- the current venue quote is fresh and the preview's size, margin, liquidation estimate and TP/SL are inspected.

If any condition is missing, the decision is **NO TRADE**.

## 5. Operating rules

- Start each review with Walt readiness, available balance, open positions, current rates and existing order/auto-close status.
- Every write follows **prepare preview → show exact terms and risk → explicit confirmation of that exact preview → execute once → reread state**.
- Never execute a trade on the basis of this note alone. A new order requires a fresh preview and separate explicit confirmation.
- After a failed or timed-out execute, reread positions and orders before deciding what to do; do not blindly retry.
- Record entry rationale, invalidation, expected costs, realized result and whether the rule was followed. Evaluate strategy quality over a sample of trades, not one outcome.

## 6. Evidence classification

- **OBSERVED:** Walt readiness, balance, positions, entry prices, sizes, leverage, reported liquidation levels, configured TP/SL triggers, latest rates and absence of pending/processing orders in the checked contracts at the stated snapshot.
- **DERIVED:** rough unrealized PnL estimates and the decision to avoid adding exposure given limited balance and incomplete candle evidence.
- **CLAIMED:** external articles' reported prices and market commentary, linked above.
- **NOT_TESTED:** any strategy's profitability, actual slippage distribution, real fill behavior of auto-close orders, or backtest performance.
- **CONFLICTED / NEEDS VERIFICATION:** cross-venue price snapshots and the material gap between TP/SL trigger values and the slippage-adjusted `price_quote` fields.

**Next action:** verify existing auto-close behavior in Walt, then obtain historical candle data and test a rules-based strategy before considering any new live position.

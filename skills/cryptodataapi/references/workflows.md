# CryptoDataAPI — worked workflows

Six copy-ready recipes. All assume `CRYPTODATA_API_KEY` is set and base
`https://cryptodataapi.com/api/v1`. Append `?format=markdown` on any GET for LLM text.

```bash
BASE=https://cryptodataapi.com/api/v1
H="X-API-Key: $CRYPTODATA_API_KEY"
```

## a) Daily market briefing  [Free]
One broad read, then two sentiment/cycle overlays.

```bash
curl -s -H "$H" "$BASE/daily?format=markdown"          # health, derivatives, macro, ETF, cycle, coins
curl -s -H "$H" "$BASE/sentiment/fear-greed"           # value, classification (Extreme Fear … Extreme Greed)
curl -s -H "$H" "$BASE/regimes/current"                # long-horizon cycle + rationale + signals
```
Expected: `/daily` → `{market_health:{total_score,long_term_score,short_term_score,state}, fear_greed, derivatives, macro, etf_flows, cycle_indicators, coins:[...], snapshot_time}`. Cross-check `market_health.state` (bear_market…topping_out) against `regimes/current.regime`; they are different-horizon lenses and may legitimately disagree.

## b) Regime → strategy selection  [Pro]
Read both regime horizons, then pull the strategy catalogue.

```bash
curl -s -H "$H" "$BASE/regimes/current"                # slow, weeks-to-months cycle read
curl -s -H "$H" "$BASE/quant/market?horizon=24h"       # fast HMM nowcast + probability buckets
curl -s -H "$H" "$BASE/trading-strategy-baskets"       # 50 baskets across 6 groups (A–F)
```
Expected: `/quant/market` → `regime{label,confidence}`, `probabilities.direction{strong_down,mild_down,flat,mild_up,strong_up}`, `probabilities.volatility/liquidation_risk/funding/breadth/open_interest`, `regime_transitions`, plus `tomorrow` (Monte-Carlo) at 24h and an `explain` block. Match the current `regime.label` to a basket's stated conditions (each basket has a stable `slug`, `name`, one-line `description`).

## c) Risk-sized position for the whole universe  [Pro]
One bulk call replaces ~2 calls per coin. Size by `vol_target_multiplier`; drop ineligible coins.

```bash
curl -s -H "$H" "$BASE/quant/coins/risk?horizon=24h"
```
Expected: `{as_of, horizon, items:[{symbol, regime{label,confidence}, vol_target_multiplier (0.25–3.0), vol_pctile_30, rv_24h, probabilities{volatility,liquidation_risk,funding,open_interest}, meta{status,insufficient_history,new_listing}}], count, universe_size}`.
Rule: `size_i = base_size * vol_target_multiplier_i`, but **skip any coin where `meta.insufficient_history` or `meta.new_listing` is true** (or `status != "ok"`) — the model hasn't seen enough bars. Directional buckets are omitted here by design; get them from `/quant/coins`.

## d) Derivatives dashboard for one coin  [Free base, Pro for positioning]
Funding + OI + liquidations, then the positioning overlay if you have Pro.

```bash
COIN=BTC
curl -s -H "$H" "$BASE/derivatives/funding-rates?coin=$COIN"      # cross-exchange funding
curl -s -H "$H" "$BASE/derivatives/open-interest?coin=$COIN"      # cross-exchange OI + trend
curl -s -H "$H" "$BASE/market-intelligence/liquidations?symbol=$COIN"   # long/short liq 1h/4h/12h/24h
curl -s -H "$H" "$BASE/quant/positioning?symbol=$COIN"           # [Pro] long/short by trader type
```
Expected: positive funding = longs pay shorts (crowded long). `open-interest` carries a 30d trend. `positioning` splits notional into `market_maker`/`whale`/`other`/`all` buckets with per-bucket account counts — the `market_maker` bucket is the basis for `/quant/gex`. Free `liquidations` is BTC-scoped; Pro extends the same call to the full universe.

## e) Backtest replay from point-in-time snapshots  [Pro Plus]
List available dates, then fetch the frozen `/daily` payload per date.

```bash
curl -s -H "$H" "$BASE/backtesting/daily-snapshots"              # -> {dates:[...]} (YYYY-MM-DD UTC)
curl -s -H "$H" "$BASE/backtesting/daily-snapshots/2026-05-28"   # full frozen /daily for that day
```
Each snapshot is identical in shape to `/daily` plus `onchain_health_score` and the frozen `technical_regime` / `signum_rgg` `by_symbol` maps — so you can regime-tag each day without look-ahead. For **bulk** multi-day history prefer `GET /backtesting/archives/download?start=YYYY-MM-DD&data_type=daily` (pre-signed files) over paginating snapshots. Deep cold tiers: `data_type=klines_deep` (1d back to 2023) and `funding_deep` (hourly funding to ~2024).

## f) Whale + gamma context before a trade  [Pro]
Layer positioning conviction, dealer gamma, and live book depth.

```bash
curl -s -H "$H" "$BASE/quant/whales"                    # ≥$100k account universe rolled up
curl -s -H "$H" "$BASE/quant/gex?symbol=SOL"            # dealer gamma / GEX analog for one coin
curl -s -H "$H" "$BASE/hyperliquid/l2-book?coin=SOL"    # live L2 bids/asks ladder
```
Expected: `/quant/whales` → `summary{net_bias(long/short/neutral), long_short_ratio, by_class}` + `top_coins[{coin, directional_net_usd, dominant_side, ...}]` (use `directional_net_usd` — net excluding MM liquidity — as the conviction read). `/quant/gex` → per coin `mm_net_delta`, `gamma_profile` (forced-unwind zones), `gamma_flip` (`null` = no meaningful flip in band, treat as "none", not zero), and a `regime` flag: `amplify` (don't fade), `dampen` (fade extremes), `transitional`. Confirm slippage against `l2-book` depth/spread before sizing.

## Conventions that apply to all of the above
- Poll `/daily` hourly, not per-tick — responses are cached (`Cache-Control` / `X-Cache`).
- On `429`, honor `Retry-After` and back off; don't hammer.
- Directional probabilities satisfy `up + down + flat = 1` — never use `1 - p_up` as fall odds.
- Watch the `X-API-Version` header to detect a response-shape change.

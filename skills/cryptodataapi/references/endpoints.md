# CryptoDataAPI — endpoint reference

Base URL `https://cryptodataapi.com/api/v1`. Every path below is relative to that.
Send `X-API-Key: cdk_live_...` on all of them except where marked **[no key]**.
Tier tags: **[Free]** = any valid key · **[Pro]** = Pro or Pro Plus · **[Pro Plus]** = Pro Plus only.
Append `?format=markdown` on most GETs for LLM-friendly text. Only endpoints present in
`llms.txt` / `llms-full.txt` are listed — nothing invented.

## Daily snapshot (start here)
- `GET /daily` — full market state in one call (health, derivatives, sentiment, macro, ETF flows, cycle indicators, coins). Params: `exchange` (hyperliquid|binance_spot|asterdex|all), `format`, `technical_detail`, `signum_detail`. [Free]
- `GET /daily/prices` — all Binance spot price pairs (~2,500). [Free]
- `GET /daily/hyperliquid` — HL perp prices, funding, OI (~229 assets). [Free]
- `GET /daily/hl-traders` — daily HL top-trader leaderboard + tracked positions. [Free]

## Coins
- `GET /coins` — coins paginated by market-cap rank. Params: `page`, `limit`. [Free]
- `GET /coins/search?q=` — search the universe by name/symbol. [Free]
- `GET /coins/top?n=` — top N by market cap. [Free]
- `GET /coins/{symbol}` — full coin profile (price, mcap, rank, supply, ATH/ATL, categories). [Free]
- `GET /coins/categories` — raw CoinGecko category tag names. [Free]
- `GET /coins/category-groups` — 20+ curated category themes with live coin lists. Param: `limit`. [Free]

## Market health
- `GET /market-health` — dual long/short score (0-100) + 11 components + indicators. Param: `format`. [Free]
- `GET /market-health/summary` — scores + sentiment + state only. [Free]
- `GET /market-health/components` — all 11 component details. [Free]
- `GET /market-health/component/{name}` — single component. [Free]
- `GET /market-health/history?days=` — daily health history (up to 730d). [Free]
- `GET /market-health/altcoin-breadth?ma_period=` — % of top alts above their MA. [Free]

## Regimes & quant engine
- `GET /regimes` — 10-state long-horizon cycle taxonomy + current detected regime + signals. [Free]
- `GET /regimes/current` — just the current long-horizon regime + rationale + signals. [Free]
- `GET /quant/regimes` — the 6-regime HMM vocabulary + per-regime basket stance. [Free]
- `GET /quant/model` — model card: version, features, walk-forward metrics, drift, runtime health. [Free]
- `GET /quant/market?horizon=4h|24h` — whole-market HMM regime + calibrated probability buckets (direction, vol, liquidation_risk, funding, breadth, OI, transitions) + Monte-Carlo `tomorrow` + `explain`. [Pro]
- `GET /quant/coins` — trimmed regime + p_direction_up/down + top_transition + OI for every HL perp (heatmap feed). Params: `horizon`, `regime`, `sort`, `limit`. [Pro]
- `GET /quant/coins/risk?horizon=` — bulk per-coin risk (regime, vol/liq/funding/OI buckets, `vol_target_multiplier`, `meta.insufficient_history`) across the whole universe in one call. [Pro]
- `GET /quant/coins/{symbol}` — full per-coin probability object conditioned on the market regime + `recent_regimes`. [Pro]
- `GET /quant/positioning?symbol=` — per-coin long/short/net/gross by trader type (market_maker/whale/other/all). [Pro]
- `GET /quant/gex?symbol=` — dealer gamma (perp GEX analog): mm_net_delta, gamma_profile, gamma_flip, amplify/dampen regime. [Pro]
- `GET /quant/whales` — the ≥$100k HL account universe rolled up: long/short notional, net_bias, top coins by conviction. [Pro]
- `GET /quant/whales/history?days=` — daily aggregate whale positioning time-series (seed→live). [Pro Plus]
- `GET /quant/history?scope=&start=` — point-in-time record of every emitted probability (json|csv). [Pro Plus]
- `GET /quant/regimes/history` — pre-signed Parquet of the full hourly 6-regime history 2020→now. [Pro Plus]
- `GET /quant/timeline?start=` — daily market regime label 2019→now, no hindsight relabeling. [Pro Plus]
- `GET /trading-strategy-baskets` — 50 trading-strategy meta-baskets across 6 groups (slug/name/description). [Pro]

## Volatility & liquidity regimes
- `GET /volatility/regime` — per-asset realized-vol regime + `vol_target_multiplier` + percentiles/z-scores. Params: `regime`, `sort`, `limit`. [Free]
- `GET /volatility/regime/score` — market-wide vol-stress composite (0-100) + `gross_exposure_multiplier`. [Free]
- `GET /volatility/regime/{symbol}` — per-asset detail + 60d vol history. [Pro]
- `GET /liquidity/depth` — per-coin L2 depth/spread snapshot across top-25 HL perps (60s refresh). [Free]
- `GET /liquidity/oi-divergence` — per-coin OI vs price divergence 1h/4h/24h. [Free]
- `GET /liquidity/depth/{coin}?minutes=` — rolling depth history (BTC on Free; full universe [Pro]). [Pro]
- `GET /liquidity/regime` — per-coin liquidity regime label + fragility score. [Pro Plus]
- `GET /liquidity/regime/score` — composite 0-100 market-structure score. [Pro Plus]

## Event-driven regimes (meme / event / security / policy)
- `GET /meme/regime` — per-asset meme lifecycle (euphoric/distribution/ignition/bleeding/dormant). [Free]
- `GET /meme/regime/score` — market-wide meme-hype composite (0-100) + meme_season flag. [Free]
- `GET /meme/regime/{symbol}` — per-asset meme detail + 60d history. [Pro]
- `GET /event/regime?window_days=` — forward catalyst calendar (unlocks, macro prints, depegs) + event-risk score. [Free]
- `GET /event/regime/score` — market-wide Event Risk composite. [Free]
- `GET /event/calendar` — queryable forward calendar. Params: `type`, `symbol`, `bias`, `min_magnitude`, `window_days`. [Free]
- `GET /event/regime/{symbol}` — per-coin pending catalysts + net bias. [Pro]
- `GET /security/regime?window_days=` — recent hacks/exploits + depegs + Security Stress score. [Free]
- `GET /security/regime/score` — market-wide Security Stress composite. [Free]
- `GET /security/events` — queryable security-events list. Params: `type`, `symbol`, `min_severity`. [Free]
- `GET /security/regime/{symbol}` — per-coin security overlay + acute bias. [Pro]
- `GET /policy/regime?window_days=` — Policy/Geopolitical regime: risk score + signed policy_tilt + FOMC catalysts. [Free]
- `GET /policy/regime/score` — market-wide Policy Risk composite + tilt. [Free]
- `GET /policy/headlines` — live classified US-regulatory headlines (pro-crypto vs restrictive). [Free]

## Indicators (Pro)
- `GET /indicators/signum-rgg` — RED/GREY/GREEN trend radar (ADX+DMI) across HL perps + top-100 Binance USDT. Params: `color`, `source`, `min_days`, `sort`, `limit`. [Pro]
- `GET /indicators/signum-rgg/{symbol}` — per-asset detail + 60d color history. [Pro]
- `GET /indicators/technical` — per-asset structure overlay: SMA-50/100/200 + cross, Bollinger squeeze, range zone, RSI(14). Params: `ma_state`, `bb`, `range_zone`, `rsi`, `sort`, `limit`. [Pro]
- `GET /indicators/technical/{symbol}` — per-asset detail + 60d history. [Pro]

## Derivatives
- `GET /derivatives/funding-rates?coin=` — cross-exchange funding (Binance + HL + aggregators). [Free]
- `GET /derivatives/open-interest?coin=` — cross-exchange OI. [Free]
- `GET /derivatives/summary?coin=` — combined cross-exchange derivatives overview. Param: `format`. [Free]
- `GET /derivatives/binance/funding-rates?symbol=&limit=` — Binance funding history. [Free]
- `GET /derivatives/binance/open-interest?symbol=` — Binance OI + 30d trend. [Free]
- `GET /derivatives/binance/long-short-ratio?symbol=` — Binance L/S account ratio. [Free]
- `GET /derivatives/binance/summary?symbol=` — all-in-one Binance derivatives. [Free]
- `GET /derivatives/binance/history?days=` — daily funding/OI/LS history (max 90d). [Free]

## Market intelligence
- `GET /market-intelligence/liquidations` — cross-exchange liquidations (long+short, 1h/4h/12h/24h). Params: `exchange`, `symbol`, `limit`. Free = BTC-scoped; [Pro] extends to full universe. [Free]
- `GET /market-intelligence/liquidations/by-exchange` — liquidations by exchange. [Free]
- `GET /market-intelligence/funding-rates` — aggregator cross-exchange funding. Params: `exchange`, `type`, `limit`. [Free]
- `GET /market-intelligence/open-interest` — aggregator cross-exchange OI. [Free]
- `GET /market-intelligence/btc/cycle-indicators?days=` — 8 BTC cycle metrics (MVRV, NUPL, Puell, Pi Cycle, …). [Free]
- `GET /market-intelligence/btc/cycle-indicators/{indicator}` — single cycle indicator. [Free]
- `GET /market-intelligence/etf/{asset}/flows` — ETF flows for btc|eth|sol|xrp. [Free]
- `GET /market-intelligence/etf/btc/aum` — BTC ETF total AUM. [Free]
- `GET /market-intelligence/options` — BTC options: OI, volume, put/call, max pain. [Free]
- `GET /market-intelligence/coinbase-premium` — Coinbase BTC premium index. [Free]
- `GET /market-intelligence/taker-buy-sell?symbol=` — taker buy/sell ratio by exchange. [Free]
- `GET /market-intelligence/borrow-interest` — margin borrow rate history. [Free]
- `GET /market-intelligence/fear-greed-history` — Fear & Greed history (~2018+). [Free]
- `GET /market-intelligence/stablecoin-history` — stablecoin mcap time-series. [Free]
- `GET /market-intelligence/grayscale/holdings` / `/grayscale/premium` — Grayscale holdings / GBTC premium. [Free]
- `GET /market-intelligence/exchange-balance` — exchange BTC balance/flow. [Free]
- `GET /market-intelligence/status` — collector status + rate-limit usage. [Free]

## Hyperliquid perps
- `GET /hyperliquid/meta` — exchange metadata (assets, max leverage, specs). [Free]
- `GET /hyperliquid/prices` — all mid prices. [Free]
- `GET /hyperliquid/open-interest` — OI across all assets. [Free]
- `GET /hyperliquid/funding-rates?coin=&limit=` — current + historical funding. [Free]
- `GET /hyperliquid/candles?coin=&interval=&limit=` — OHLCV candles. [Free]
- `GET /hyperliquid/l2-book?coin=` — L2 order-book snapshot (BTC on Free; full universe [Pro]). [Free]
- `GET /hyperliquid/summary?coin=` — price + funding + OI + recent candles. [Free]

## Hyperliquid trader intelligence
- `GET /hyperliquid/top-traders` — scored top-trader leaderboard. [Free]
- `GET /hyperliquid/wallet-positions?address=` — live positions for any HL address. [Free]
- `GET /hyperliquid/wallet-signals?address=&minutes=` — entry/exit/size-change signals. [Free]
- `GET /hyperliquid/trader-profile/{address}?days=` — on-demand win rate/PnL/edges for any address. [Free]
- `GET /hyperliquid/wallet-trades/{address}?days=` — historical trades + PnL for any address. [Free]
- `GET /hyperliquid/watchlist` — list watchlisted wallets. `POST`/`DELETE` mutate ([Pro]). [Free]
- `GET /hyperliquid/wallets/search` — filter the leaderboard by performance gates. [Pro]
- `GET /hyperliquid/copy-signals` — top traders + their recent signals in one call. [Pro]

## On-chain intelligence
- `GET /on-chain/score` — composite 0-100 on-chain health (MVRV, Hash Ribbon, dry-powder, whales, flows, miners). [Free]
- `GET /on-chain/exchange-flows/{symbol}` — net CEX flow (summed + `by_chain`), 1h/6h/24h/7d, spike z-scores. Param: `exchange`. [Free]
- `GET /on-chain/exchange-flows/spike-alerts?min_amount=` — large-transfer feed (default $1M). [Free]
- `GET /on-chain/stablecoin-reserves` — CEX USDT/USDC balances across ETH/Tron/BSC. [Free]
- `GET /on-chain/stablecoin-reserves/dry-powder` — z-scored accumulating/neutral/depleting signal. [Free]
- `GET /on-chain/miners/reserves` — BTC miner pool reserves + 24h/7d/30d net flow. [Free]
- `GET /on-chain/miners/hash-ribbon` — Hash Ribbon capitulation/recovery state. [Free]
- `GET /on-chain/dormancy/btc` — MVRV + zone + active addresses + chain-wide exchange flow. [Free]
- `GET /on-chain/whales` — 🚧 top-100 non-CEX holder snapshot (temporarily disabled). [Free]

## Sentiment & macro
- `GET /sentiment/fear-greed` — multi-source Fear & Greed (0-100) + classification. Param: `format`. [Free]
- `GET /sentiment/macro` — EUR/USD, gold, treasury yields, DXY. [Free]
- `GET /sentiment/stablecoins` — stablecoin mcap + 14d/90d flows (liquidity gauge). [Free]
- `GET /sentiment/stablecoins/history` / `/remote-history?days=` — stablecoin mcap time-series. [Free]

## Market data (Binance spot)
- `GET /market-data/klines?symbol=&interval=&limit=` — OHLCV klines (max 1000). [Free]
- `GET /market-data/ticker/24hr?symbol=` / `/ticker/price?symbol=` — 24h stats / current price. [Free]
- `GET /market-data/btc-price-history?days=` — BTC price + 200D MA. [Free]
- `GET /market-data/volume-history?days=` — daily volume + buy ratio. [Free]

## DEX & meme coins
- `GET /dex/trending?chain=` — trending DEX pools across chains (2-min refresh). [Free]
- `GET /dex/new-pools?chain=` — newest DEX pools. [Free]
- `GET /dex/token/{chain}/{address}` — token info + top pools. [Free]
- `GET /dex/promoted` / `/promoted/top` — promoted/boosted tokens (marketing-spend signal). [Free]
- `GET /dex/security/{chain}/{address}` — token security report (honeypot/mint/tax + risk score). [Free]

## NFT trade volume (Pro)
- `GET /nfts/overview?days=` — daily NFT volume: totals + by chain/tier/category + top collections + marketplaces. [Pro]
- `GET /nfts/volume?by=chain|tier|category|collection` — daily volume buckets, date-bounded. [Pro]
- `GET /nfts/collections` / `/collections/{slug}?days=` — seeded taxonomy / per-collection series. [Pro]
- `GET /nfts/correlations?vs=btc,eth,sol&window_days=` — NFT-volume vs asset-return Pearson correlation. [Pro Plus]

## Backtesting & historical (Pro Plus, isolated bulkhead)
- `GET /backtesting/daily-snapshots` — list of dates with an archived `/daily` snapshot. [Pro Plus]
- `GET /backtesting/daily-snapshots/{date}` — one day's full point-in-time snapshot (frozen regime/health). [Pro Plus]
- `GET /backtesting/snapshots?data_type=&start=&limit=` — streamed time-series JSON for a data_type (coinglass_oi/funding, market_health, fear_greed, gamma_exposure, liquidation_map, hl_l2_books, …). [Pro Plus]
- `GET /backtesting/snapshots/types` — list all snapshot data_types + coverage ranges. [Pro Plus]
- `GET /backtesting/klines?symbol=&exchange=&start=` — historical 1m OHLCV (json|csv). [Pro Plus]
- `GET /backtesting/funding?symbol=&start=` — historical funding + OI. [Pro Plus]
- `GET /backtesting/liquidations?start=` — historical liquidation events. [Pro Plus]
- `GET /backtesting/symbols` / `/status` — tracked symbols + ranges / storage stats. [Pro Plus]
- `GET /backtesting/archives` / `/archives/index` — list archived cold-storage files (data_type incl. `daily`, `klines_deep`, `funding_deep`). [Pro Plus]
- `GET /backtesting/archives/download?start=&data_type=` — pre-signed bulk-download URLs (recommended for multi-day). [Pro Plus]
- `GET /backtesting/export?symbol=&start=` — streaming CSV export (up to 100k rows). [Pro Plus]

## Payments, auth, health
- `GET /payments/plans` — plan pricing + networks. **[no key]**
- `POST /payments/agent-subscribe` — gasless x402 agent onboarding (see tiers reference). [Free]
- `POST /payments/subscribe` — subscribe to a plan via x402 (optional `discount_code`). [Free]
- `GET /payments/subscription` / `/invoices` — subscription status / invoices. [Free]
- `POST /auth/keys` — create a free key `{"email":...}`. **[no key]**
- `GET /auth/keys/me` · `POST /auth/keys/rotate` · `DELETE /auth/keys` — manage the current key. [Free]
- `GET /health` — basic liveness. **[no key]**
- `GET /changelog` — public JSON changelog of notable response-shape changes. **[no key]**

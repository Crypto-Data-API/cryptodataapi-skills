---
name: cryptodataapi
description: Live crypto market data and analysis via CryptoDataAPI (cryptodataapi.com) — market regimes, funding, open interest, liquidations, whale activity, dealer gamma, on-chain flows, sentiment, a quant risk engine, and historical backtesting. Use when the user wants live crypto market data, to read the market regime, analyze derivatives, build/size/evaluate a trading strategy, or backtest against historical snapshots.
---

# CryptoDataAPI — crypto market analysis & backtesting playbook

Use the CryptoDataAPI REST API (base `https://cryptodataapi.com/api/v1`) for live crypto
market data, market regimes, derivatives, on-chain flows, a quant risk engine, and
backtesting — one key, 180+ endpoints, 12 aggregated sources. If the `cryptodataapi`
**MCP** tools are loaded, prefer those tool calls; otherwise call the REST endpoints
directly (curl / requests) with the user's key.

## Auth
- Header on every request: `X-API-Key: cdk_live_...`. Read the key from the
  `CRYPTODATA_API_KEY` environment variable. (`/health` and `/payments/plans` need no key.)
- No key yet? Create a free one, no signup: `POST /api/v1/auth/keys` with
  `{"email":"you@example.com"}` — the full key is returned **once**.

```bash
curl -H "X-API-Key: $CRYPTODATA_API_KEY" https://cryptodataapi.com/api/v1/daily
```

- **Tiers** (every tier is a complete product, not a teaser):
  - **Free** — the whole market-wide picture: health scores, regime reads, Fear & Greed,
    macro, ETF & stablecoin flows, BTC cycle indicators, 2,000+ coin profiles, and
    full-universe funding + open interest across all Hyperliquid perps, plus BTC-scoped
    order books & liquidations. 5 req/min, 50/day.
  - **Pro** — from $39/mo annual ($49/mo monthly). Adds the proprietary per-coin signals
    for **every** coin: quant regime & forward probabilities, dealer gamma (GEX), whale
    activity, trader positioning, full-universe liquidations / L2 books, trading indicators
    and strategy baskets. 30 req/min, 10,000/day.
  - **Pro Plus** — from $149/mo annual ($199/mo monthly). Adds the historical & bulk layer:
    the 2020→now regime archive (Parquet), a probability audit trail, and backtesting data.
    60 req/min, unlimited daily.
- Agents can subscribe autonomously with gasless USDC via
  `POST /api/v1/payments/agent-subscribe` (x402: 402 → sign off-chain → settle → key).
  See `references/tiers-and-limits.md` for plan slugs and the flow.

## Always start here
`GET /api/v1/daily` — one cached call bundling market health, derivatives, sentiment,
macro, ETF flows and cycle indicators. It replaces ~10 calls. Append `?format=markdown`
to most endpoints for clean LLM context. Poll hourly, not per-tick.

## The core loop: regime → strategy → risk → backtest
1. **Read the regime** — `GET /regimes/current` (10-state long-horizon cycle) and
   `GET /quant/market` (HMM market regime + calibrated probability buckets). *(quant = Pro)*
2. **Pick strategies** — `GET /trading-strategy-baskets` (50 meta-baskets across 6 groups,
   each pre-mapped to the conditions it was built for). Use `GET /regimes` for the taxonomy. *(Pro)*
3. **Size the risk** — `GET /quant/coins/risk?horizon=24h` returns, in one bulk call for the
   whole universe, each coin's regime, the volatility / liquidation_risk / funding /
   open_interest probability buckets, and a `vol_target_multiplier` (0.25–3.0). Honor
   `meta.insufficient_history` / `meta.new_listing` — mark those coins ineligible. *(Pro)*
4. **Backtest** — `GET /backtesting/daily-snapshots` (available dates) and
   `GET /backtesting/daily-snapshots/{date}` (the full point-in-time `/daily` payload for that
   day, incl. frozen regime/health). Replay history to validate before going live. *(Pro Plus)*

## Go deeper
- **Full endpoint reference** (every domain, one line + tier per endpoint): `references/endpoints.md`
- **Worked workflows** (copy-ready curl, expected fields): `references/workflows.md`
- **Tiers, rate limits, errors, x402 self-subscribe, MCP**: `references/tiers-and-limits.md`

## Conventions
- **Respect the cache.** Responses carry `Cache-Control` / `X-Cache`; data refreshes on
  fixed cadences (1–30 min). Polling faster just wastes quota.
- **Batch, don't fan out.** Use `/quant/coins` and `/quant/coins/risk` for the whole
  universe in one call instead of per-symbol loops.
- **Handle rate limits.** `429` (with `Retry-After`) = back off. Free 5/min · Pro 30/min ·
  Pro Plus 60/min.
- **up + down + flat = 1** for directional probabilities — never treat `1 - p_up` as fall odds.
- **Watch `X-API-Version`** (CalVer) to detect a response-shape change without diffing; the
  public JSON `/changelog` records notable ones.

## References
- Machine-readable map: https://cryptodataapi.com/llms.txt · full: https://cryptodataapi.com/llms-full.txt
- Docs: https://cryptodataapi.com/api/docs · OpenAPI: https://cryptodataapi.com/api
- Agent quickstart: https://cryptodataapi.com/ai-agents
- **MCP alternative:** if the `cryptodataapi` MCP tools are loaded, prefer them (they call
  these same endpoints with your key). Remote install (with the key exported in the shell,
  never pasted into a chat — see references/tiers-and-limits.md "Key hygiene"):
  `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header 'X-API-Key: ${CRYPTODATA_API_KEY}'`

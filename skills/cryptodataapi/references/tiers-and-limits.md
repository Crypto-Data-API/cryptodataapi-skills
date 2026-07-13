# CryptoDataAPI — tiers, limits, errors & agent payments

## Tier matrix

Every tier is a complete product on its own, not a locked teaser. Free already covers the
whole **market-wide** picture; Pro unlocks the **per-coin** proprietary signals for *every*
coin; Pro Plus adds the **historical & bulk** layer.

| | **Free** | **Pro** | **Pro Plus** |
|---|---|---|---|
| Price | $0, no card | from **$39/mo** annual ($468/yr) · $49/mo monthly | from **$149/mo** annual ($1,788/yr) · $199/mo monthly |
| Rate limit | 5 req/min, 50/day | 30 req/min, 10,000/day | 60 req/min, unlimited/day |
| Market-wide data | health scores, regime reads, Fear & Greed, macro, ETF & stablecoin flows, BTC cycle indicators, 2,000+ coin profiles & prices, full-universe funding + OI across all HL perps | everything in Free | everything in Pro |
| Order books / liquidations | **BTC-scoped** only | **full-universe** | full-universe |
| Per-coin quant | — | quant regime & forward probabilities, dealer gamma (GEX), whale activity, trader positioning, per-coin vol/meme/event/security overlays | same |
| Indicators & strategy | — | SIGNUM RGG + technical/structural indicators, 50 strategy baskets, NFT trends | same |
| Historical & bulk | — | — | 2020→now regime archive (Parquet), point-in-time probability audit trail, backtesting snapshots/klines/funding/liquidations + archive downloads |

Prices verified against `clients/mcp-npm/README.md` and `templates/pricing.html`
(annual "from" rate is the lower figure; monthly is higher). Pro annual saves ~$120/yr,
Pro Plus annual saves ~$600/yr. Cancel-anytime with a 7-day money-back guarantee.

## Rate limits & 429 handling
- Free **5/min · 50/day** · Pro **30/min · 10,000/day** · Pro Plus **60/min · unlimited/day**.
- Over the limit → HTTP **429** with a `Retry-After` header. Nothing breaks — honor
  `Retry-After` and back off. Do not retry in a tight loop.
- Free's 50/day is a daily cap too; a long session should lean on `/daily` (one call ≈ 10
  endpoints) rather than many small reads.

## Caching & cadences
- Responses carry `Cache-Control` and an `X-Cache` (hit/miss) header. Data refreshes on
  fixed cadences (roughly 1–30 min depending on the feed; DEX trending ~2 min, quant ~15 min,
  most regime caches 30 min, on-chain 5–30 min). Polling faster than the cadence just wastes
  quota and returns the same bytes.
- `/daily` is a bundled cached snapshot — poll it **hourly**, not per-tick.
- Watch the `X-API-Version` header (CalVer) to detect a response-shape change without diffing
  payloads; the public JSON `GET /changelog` (no key) records notable ones.

## `?format=markdown`
Most GET endpoints accept `?format=markdown` and return clean plain-text instead of JSON —
cheaper to drop straight into an LLM context window. Supported on `/daily`, `/market-health`,
`/sentiment/fear-greed`, `/derivatives/summary`, and more.

## Agent self-subscribe (x402, gasless USDC)
Agents can pay and upgrade autonomously — no dashboard, no gas — over the
[x402](https://x402.org) protocol:

```bash
curl -s -X POST -H "X-API-Key: $CRYPTODATA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"plan":"monthly"}' \
  https://cryptodataapi.com/api/v1/payments/agent-subscribe
```

Flow: the endpoint returns **HTTP 402** with payment requirements → the agent signs an
off-chain USDC authorization → the server settles it on-chain → an upgraded key is returned.
Plan slugs: `monthly` (Pro $49) · `annual` (Pro $468/yr) · `monthly_plus` (Pro Plus $199) ·
`annual_plus` (Pro Plus $1,788/yr). See https://cryptodataapi.com/ai-agents for the full flow.
`GET /payments/plans` (no key) lists live pricing + supported networks.

## Common errors
- **401** — missing/invalid key. Set `X-API-Key: cdk_live_...`; mint a free one with
  `POST /api/v1/auth/keys` `{"email":"you@example.com"}` (key shown once).
- **403** — tier gate. The endpoint needs Pro or Pro Plus. Upgrade via the x402 flow above
  or https://cryptodataapi.com/pricing, then retry. The error body names the required tier.
- **429** — rate limited. Honor `Retry-After`, back off (see above).
- **503** — a feed is warming (e.g. quant just after a deploy, or a leaderboard first refresh).
  Retry after a short delay; `GET /quant/model` (`loaded`) tells you if the quant model is up.

## MCP alternative
If the user runs an MCP client (Claude, Cursor, Windsurf…), the hosted MCP server exposes
these same endpoints as native tools (same key, same auth/tiers/rate-limits — it calls the
REST API in-process). Prefer the `cryptodataapi` MCP tools when they are loaded.

- **Remote (recommended, no Node).** Export the key once in the shell profile, then use the
  `${CRYPTODATA_API_KEY}` placeholder — single-quoted so it is stored literally; Claude Code
  expands it from the environment at connect time. The command then contains no secret (safe
  to paste anywhere, including a chat) and the plaintext never lands in `~/.claude.json`:
  ```bash
  export CRYPTODATA_API_KEY="cdk_live_YOUR_KEY"   # terminal only
  claude mcp add --transport http cryptodataapi \
    https://cryptodataapi.com/mcp \
    --header 'X-API-Key: ${CRYPTODATA_API_KEY}'
  ```
- **Stdio (npm bridge)** — reads the same env var:
  ```bash
  claude mcp add cryptodataapi -- npx -y cryptodataapi-mcp
  ```

**Key hygiene:** never paste a command containing a literal `cdk_live_` key into an AI chat —
it persists in that conversation's logs. If a key leaks, rotate it at
https://cryptodataapi.com/dashboard.

Tools include `get_daily_snapshot`, `get_market_health`, `get_market_regime`,
`get_funding_rates`, `get_open_interest`, `get_liquidations`, `get_whale_activity`,
`get_gamma_exposure`, `get_positioning`, `create_free_api_key`, plus a `query_api` escape
hatch for any other GET endpoint. The heavy `/backtesting/*` routes are intentionally not MCP
tools (isolated bulkhead) — call those over REST.

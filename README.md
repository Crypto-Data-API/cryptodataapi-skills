# CryptoDataAPI Agent Skills

An installable [Agent Skill](https://docs.claude.com/en/docs/claude-code/skills) that teaches an AI agent to use **[CryptoDataAPI](https://cryptodataapi.com)** — live crypto market data, market regimes, funding & open interest, liquidations, whale activity, dealer gamma, on-chain flows, a quant risk engine, and historical backtesting, all behind one API key.

> Live crypto market data and analysis via CryptoDataAPI (cryptodataapi.com) — market regimes, funding, open interest, liquidations, whale activity, dealer gamma, on-chain flows, sentiment, a quant risk engine, and historical backtesting. Use when the user wants live crypto market data, to read the market regime, analyze derivatives, build/size/evaluate a trading strategy, or backtest against historical snapshots.

The skill ships a lean `SKILL.md` playbook plus progressive-disclosure reference files (full endpoint map, worked workflows, tiers/limits) that the agent loads only when it needs them.

## Install

**1. skills.sh CLI (recommended)**

```bash
npx skills add Crypto-Data-API/cryptodataapi-skills -g -y
```

Installs the `cryptodataapi` skill globally (`-g`) without prompts (`-y`).

**2. claude.ai zip upload**

Download `dist/cryptodataapi-skill.zip` from this repo, then in claude.ai go to **Settings → Capabilities → Skills** and upload the zip. It unpacks to a single `cryptodataapi/` skill folder.

**3. Manual (git clone)**

```bash
git clone https://github.com/Crypto-Data-API/cryptodataapi-skills.git
cp -r cryptodataapi-skills/skills/cryptodataapi ~/.claude/skills/
```

Copy `skills/cryptodataapi` into your agent's skills directory (e.g. `~/.claude/skills/` for Claude Code).

## What's inside

```
skills/
  cryptodataapi/
    SKILL.md
    references/
      endpoints.md
      tiers-and-limits.md
      workflows.md
```

## Set your API key

The skill reads the `CRYPTODATA_API_KEY` environment variable and sends it as the `X-API-Key` header. No key yet? Mint a free one (no signup):

```bash
curl -X POST https://cryptodataapi.com/api/v1/auth/keys \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com"}'
```

## Tiers

- **Free** — the whole market-wide picture: health scores, regime reads, Fear & Greed, macro, ETF & stablecoin flows, BTC cycle indicators, 2,000+ coin profiles, full-universe funding + open interest, and BTC-scoped order books & liquidations. 5 req/min.
- **Pro** (from $39/mo annual, $49/mo monthly) — the per-coin proprietary signals for every coin: quant regime & forward probabilities, dealer gamma (GEX), whale activity, positioning, full-universe liquidations / L2 books, indicators, strategy baskets. 30 req/min.
- **Pro Plus** (from $149/mo annual, $199/mo monthly) — the historical & bulk layer: 2020→now regime archive (Parquet), a probability audit trail, and backtesting data. 60 req/min.

Agents can subscribe autonomously over x402 (gasless USDC) — see the [agent quickstart](https://cryptodataapi.com/ai-agents).

## Links

- Agent Skill page: https://cryptodataapi.com/ai-agents/agent-skill
- API docs: https://cryptodataapi.com/api/docs
- Machine-readable map: https://cryptodataapi.com/llms.txt
- MCP server (npm): https://www.npmjs.com/package/cryptodataapi-mcp

---

_Generated from the `cryptodataapi` repo's `content/skill/` — **do not edit by hand**. PRs and issues → the main repo, which regenerates this mirror._

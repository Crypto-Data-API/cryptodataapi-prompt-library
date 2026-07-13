# MCP Market Analyst (Claude Desktop / Cursor)

> Wire CryptoDataAPI into Claude through an MCP tool so Claude can pull a full one-call market snapshot on demand and answer any market question grounded in live data — no copy-paste, no stale context.

## Use Case
Wire CryptoDataAPI into Claude through an MCP tool so Claude can pull a full one-call market snapshot on demand and answer any market question grounded in live data — no copy-paste, no stale context.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/daily` (Free tier)
- **Fields used:** `daily`, `market_health`, `sentiment`, `derivatives`

## The Prompt
````text
[SYSTEM]
You are a crypto market analyst running inside an MCP client (Claude Desktop, Claude Code, Cursor, Continue...) with the cryptodataapi MCP server connected — install: claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp (key via header; see cryptodataapi.com/ai-agents/mcp-server).

YOUR PRIMARY TOOL: get_daily_snapshot — CryptoDataAPI's one-call market snapshot: market_health (total/long_term/short_term scores + indicators like breadth_pct, pct_from_200ma, cross_status), fear_greed {value, classification}, btc_derivatives (funding.current_rate/avg_rate, open_interest.trend_30d_pct, long_short ratios), macro, stablecoins, ETF flows, regime overlays, and snapshot_built_at. Call it at the START of answering ANY question about current market state; once per user turn, reuse within the turn. No MCP tools available? Fall back to GET https://cryptodataapi.com/api/v1/daily with the X-API-Key header.

GROUNDING RULES (non-negotiable):
- ALWAYS ground market answers in the snapshot. CITE the fields you used inline, e.g. (market_health.long_term_score), (fear_greed.value), (btc_derivatives.funding.current_rate). Do not invent numbers the payload does not contain.
- IMPORTANT: the snapshot is a DAILY build (rebuilds ~20:00 UTC + on server start — check snapshot_built_at), not live tick data. State its age and never present it as a real-time price feed.
- Surface DIVERGENCES explicitly — e.g. net-long positioning into a fearful, structurally weak tape is a crowding risk worth its own line.
- Explain, contextualise, and flag risk. Do not give financial advice, price targets, or buy/sell calls.
- If a field the user asks about is absent from the snapshot, say so plainly rather than guessing.

TERMINAL VISUALS: alongside your table, include a compact at-a-glance dashboard inside a fenced code block — quantized Unicode bars render perfectly in terminals and monospace chat views:
- 0-100 scores as 10-block meters (1 block = 10, round): 'Health     32/100  ███▒▒▒▒▒▒▒'
- Probability/share bars, one █ per ~4%, value at the end: 'flat       █████████ 36%'
- Short series as sparklines ▁▂▃▄▅▆▇█ (min-max scaled): 'health 30d ▆▅▄▃▃▂▂▃▂▁'
- Signed values around a │ axis: '  ◀██ -0.9%  │  +2.1% ████▶'
- Status glyphs: ↑ ↓ → ● ○
Align columns with spaces, quantize honestly (never imply precision the data lacks), keep the dashboard under ~12 lines.
Chart here: meters for market-health total / long-term / short-term and Fear & Greed, a funding line (current vs average with ↑↓), and a long/short positioning bar.

[USER]
Call the cryptodataapi get_daily_snapshot MCP tool (fallback: GET https://cryptodataapi.com/api/v1/daily with the X-API-Key header from the CRYPTODATA_API_KEY env var). If a payload is already pasted below this prompt, use that instead.

Using ONLY this snapshot, give me a grounded read of the market right now. Return a markdown table with columns Dimension | Reading | Field | What it means, covering: market health (total + long-term vs short-term), breadth/structure (breadth_pct, pct_from_200ma, cross_status), Fear & Greed, funding (current vs average), open interest trend, and long/short positioning. Follow the table with a 2-3 sentence Net paragraph that names any divergence between the dimensions, then one line stating the snapshot's build time and age (once-a-day snapshot, not real-time). No trade advice.

[OUTPUT FORMAT — mimic the structure, not the values]
| Dimension | Reading | Field | What it means |
|-----------|---------|-------|---------------|
| Market health | 32/100 - BEARISH | market_health.total_score / .sentiment | Structurally weak tape |
| LT vs ST | 19 vs 45 | market_health.long_term_score / .short_term_score | Structural leg is the damage; near-term merely less bad |
| Breadth / structure | 13% above MA, -14% vs 200DMA, Death cross | indicators.breadth_pct / .pct_from_200ma / .cross_status | Broad, confirmed downtrend |
| Fear & Greed | 28 - Fear | fear_greed.value / .classification | Risk-off, not capitulation |
| Funding | +0.0015%/8h vs +0.0069% avg | btc_derivatives.funding | Longs pay, but fading - de-leveraging |
| Open interest | +0.5% / 30d | btc_derivatives.open_interest.trend_30d_pct | No leverage build |
| Positioning | 64/36 long, ratio 1.78 (top traders 1.96) | btc_derivatives.long_short | Crowd net long into weakness |

Net: a structurally bearish backdrop (LT 19, death cross, thin breadth) with Fear sentiment and de-leveraging derivatives - yet positioning skews net long, a crowding divergence worth watching rather than a confirmation.

Snapshot built 2026-07-13 03:28 UTC (~40 min old) - a once-daily snapshot, not real-time data. Not financial advice.

```
Health      32/100  ███▒▒▒▒▒▒▒  BEARISH
Long-term   19/100  ██▒▒▒▒▒▒▒▒  structural damage
Short-term  45/100  █████▒▒▒▒▒  less bad, not recovery
Fear&Greed  28/100  ███▒▒▒▒▒▒▒  Fear
Funding     +0.0015%/8h ↓ (avg +0.0069%)  de-leveraging
L/S         long ██████░░░ short  64/36 (crowded long ⚠)
```
````

## Example Output
````
| Dimension | Reading | Field | What it means |
|-----------|---------|-------|---------------|
| Market health | 32/100 - BEARISH | market_health.total_score / .sentiment | Structurally weak tape |
| LT vs ST | 19 vs 45 | market_health.long_term_score / .short_term_score | Structural leg is the damage; near-term merely less bad |
| Breadth / structure | 13% above MA, -14% vs 200DMA, Death cross | indicators.breadth_pct / .pct_from_200ma / .cross_status | Broad, confirmed downtrend |
| Fear & Greed | 28 - Fear | fear_greed.value / .classification | Risk-off, not capitulation |
| Funding | +0.0015%/8h vs +0.0069% avg | btc_derivatives.funding | Longs pay, but fading - de-leveraging |
| Open interest | +0.5% / 30d | btc_derivatives.open_interest.trend_30d_pct | No leverage build |
| Positioning | 64/36 long, ratio 1.78 (top traders 1.96) | btc_derivatives.long_short | Crowd net long into weakness |

Net: a structurally bearish backdrop (LT 19, death cross, thin breadth) with Fear sentiment and de-leveraging derivatives - yet positioning skews net long, a crowding divergence worth watching rather than a confirmation.

Snapshot built 2026-07-13 03:28 UTC (~40 min old) - a once-daily snapshot, not real-time data. Not financial advice.

```
Health      32/100  ███▒▒▒▒▒▒▒  BEARISH
Long-term   19/100  ██▒▒▒▒▒▒▒▒  structural damage
Short-term  45/100  █████▒▒▒▒▒  less bad, not recovery
Fear&Greed  28/100  ███▒▒▒▒▒▒▒  Fear
Funding     +0.0015%/8h ↓ (avg +0.0069%)  de-leveraging
L/S         long ██████░░░ short  64/36 (crowded long ⚠)
```
````

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/daily
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- One call, whole picture: get_daily_snapshot (REST: /api/v1/daily) bundles prices, market_health, sentiment, derivatives, macro, flows and the regime overlays, so a single tool covers most market Q&A without chaining endpoints.
- This uses the REAL cryptodataapi MCP server (24 tools) - no hand-registered tool needed. One-line install on the MCP Server docs page: cryptodataapi.com/ai-agents/mcp-server. /api/v1/daily works on any key (free tier).
- It is a daily snapshot (rebuilds ~20:00 UTC + on server start), so cache it for the day - polling more often just returns the same payload. Add ?format=markdown on the REST path for LLM-friendly output.
- Follow-up drill-downs pair well: get_market_regime for the HMM read, get_funding_rates / get_open_interest for per-coin derivatives, get_fear_greed for sentiment history.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5_

# Funding Rate Extremes Scanner

> Surface coins whose perpetual funding has gone extreme, flagging crowded, over-leveraged positioning that often precedes a squeeze.

## Use Case
Surface coins whose perpetual funding has gone extreme, flagging crowded, over-leveraged positioning that often precedes a squeeze.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/market-intelligence/funding-rates` (Free tier)
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/market-intelligence/open-interest` (Free tier)
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/derivatives/funding-rates` (Free tier)
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/hyperliquid/funding-rates` (Free tier)
- **Fields used:** `funding_rate`, `open_interest`, `cross_exchange`, `coin`

## The Prompt
```text
[SYSTEM]
You are an expert crypto derivatives analyst. You interpret perpetual-swap funding rates as a real-time gauge of leveraged positioning: persistently positive funding means longs are paying shorts (crowded long, squeeze risk to the downside), and negative funding means shorts are paying longs (crowded short, squeeze risk to the upside).

Rules:
- Annualise funding where useful (perp funding is typically charged every 8h -> x3 per day x365).
- Thresholds apply to the 8h rate, NOT the annualised figure: |8h funding| > 0.05% (~55%/yr) is elevated, > 0.10% (~110%/yr) is extreme.
- Weight your read by open interest: extreme funding on large OI matters far more than on a thin market. If a coin has no OI coverage, say so and mark its read low-confidence.
- A calm tape is a finding, not a failure: if nothing crosses the elevated threshold, state plainly that leverage is light market-wide and show the most RELATIVELY stretched names for context.
- When you pull per-coin funding history for momentum, always confirm the response echoes the coin/symbol you asked for - the endpoints default to BTC when the parameter is omitted or misspelled.
- Never give financial advice. Describe positioning and risk, not 'buy' or 'sell' calls.

[USER]
First, get the live data: GET https://cryptodataapi.com/api/v1/market-intelligence/funding-rates ; GET https://cryptodataapi.com/api/v1/market-intelligence/open-interest ; GET https://cryptodataapi.com/api/v1/derivatives/funding-rates ; GET https://cryptodataapi.com/api/v1/hyperliquid/funding-rates — auth with the X-API-Key header (key in the CRYPTODATA_API_KEY env var), or use the cryptodataapi MCP tools. If a payload is already pasted below this prompt, use that instead; if you cannot make network calls, ask me to paste it.

Identify the coins with the most extreme funding (both positive and negative). For each, state: the 8h and annualised rate, whether longs or shorts are crowded, the open interest context, and the direction of squeeze risk. Rank them by how stretched the positioning is, and call out any coin where funding and OI are BOTH rising (building leverage). Return a short markdown table plus a two-sentence summary.

[OUTPUT FORMAT — mimic the structure, not the values]
| Coin | 8h funding | Annualised | Crowded side | OI context | Squeeze risk |
|------|-----------|-----------|-------------|-----------|-------------|
| SOL  | +0.11%    | +120%     | Longs       | OI +22% / 24h | Downside |
| PEPE | -0.09%    | -98%      | Shorts      | OI flat       | Upside   |

Summary: Positioning is most stretched in SOL, where longs are paying a triple-digit annualised rate into rising open interest — a classic crowded-long setup vulnerable to a long squeeze. PEPE shorts are similarly crowded but on flat OI, a weaker signal.
```

## Example Output
```
| Coin | 8h funding | Annualised | Crowded side | OI context | Squeeze risk |
|------|-----------|-----------|-------------|-----------|-------------|
| SOL  | +0.11%    | +120%     | Longs       | OI +22% / 24h | Downside |
| PEPE | -0.09%    | -98%      | Shorts      | OI flat       | Upside   |

Summary: Positioning is most stretched in SOL, where longs are paying a triple-digit annualised rate into rising open interest — a classic crowded-long setup vulnerable to a long squeeze. PEPE shorts are similarly crowded but on flat OI, a weaker signal.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/market-intelligence/funding-rates
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- Universe scan: /api/v1/market-intelligence/funding-rates returns per-exchange 8h funding for the full coin universe (~177 coins); /api/v1/market-intelligence/open-interest adds all-exchange OI with 24h change for the top ~25 - coins outside that set have no OI context, so weight-by-OI reads on them are low-confidence.
- Single-coin drill-down: /api/v1/derivatives/funding-rates?coin=X gives the cross-exchange snapshot; funding momentum needs history - /api/v1/hyperliquid/funding-rates?coin=X or /api/v1/derivatives/binance/funding-rates?symbol=XUSDT. Check the echoed coin/symbol field matches what you requested.
- Pair with /api/v1/quant/market for regime context - extreme funding in a strong-trend regime resolves differently than in a chop regime.
- Add ?format=markdown on supported endpoints for LLM-friendly plain-text output you can hand your LLM clean plain text instead of raw JSON.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

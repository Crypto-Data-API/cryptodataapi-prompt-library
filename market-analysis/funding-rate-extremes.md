# Funding Rate Extremes Scanner

> Surface coins whose perpetual funding has gone extreme, flagging crowded, over-leveraged positioning that often precedes a squeeze.

## Use Case
Surface coins whose perpetual funding has gone extreme, flagging crowded, over-leveraged positioning that often precedes a squeeze.

## Data Required
- **Endpoint:** `GET /api/v1/derivatives/funding-rates` (Free tier)
- **Endpoint:** `GET /api/v1/hyperliquid/summary` (Free tier)
- **Fields used:** `funding_rate`, `open_interest`, `cross_exchange`, `coin`

## The Prompt
```text
[SYSTEM]
You are an expert crypto derivatives analyst. You interpret perpetual-swap funding rates as a real-time gauge of leveraged positioning: persistently positive funding means longs are paying shorts (crowded long, squeeze risk to the downside), and negative funding means shorts are paying longs (crowded short, squeeze risk to the upside).

Rules:
- Annualise funding where useful (perp funding is typically charged every 8h → x3 per day x365).
- Treat |8h funding| > 0.05% as elevated and > 0.10% as extreme.
- Weight your read by open interest: extreme funding on large OI matters far more than on a thin market.
- Never give financial advice. Describe positioning and risk, not 'buy' or 'sell' calls.

[USER]
Here is the current cross-exchange funding data from CryptoDataAPI:

{data}

Identify the coins with the most extreme funding (both positive and negative). For each, state: the 8h and annualised rate, whether longs or shorts are crowded, the open interest context, and the direction of squeeze risk. Rank them by how stretched the positioning is, and call out any coin where funding and OI are BOTH rising (building leverage). Return a short markdown table plus a two-sentence summary.
```

## Example Output
```
| Coin | 8h funding | Annualised | Crowded side | OI context | Squeeze risk |
|------|-----------|-----------|-------------|-----------|-------------|
| SOL  | +0.11%    | +120%     | Longs       | OI +22% / 24h | Downside |
| PEPE | -0.09%    | -98%      | Shorts      | OI flat       | Upside   |

Summary: Positioning is most stretched in SOL, where longs are paying a triple-digit annualised rate into rising open interest — a classic crowded-long setup vulnerable to a long squeeze. PEPE shorts are similarly crowded but on flat OI, a weaker signal.
```

## Notes
- Pair with /api/v1/quant/market for regime context — extreme funding in a strong-trend regime resolves differently than in a chop regime.
- Cross-exchange rows expose binance{}, hyperliquid{} and a cross_exchange summary so you can spot venue divergence.
- Add ?format=markdown on the endpoint for LLM-friendly plain-text output you can paste straight into {data}.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

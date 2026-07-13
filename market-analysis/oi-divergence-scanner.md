# Open Interest Divergence Scanner

> Find coins where price and open interest are diverging - revealing the conviction behind a move, from fresh-money trends to hollow short-covering bounces.

## Use Case
Find coins where price and open interest are diverging - revealing the conviction behind a move, from fresh-money trends to hollow short-covering bounces.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/liquidity/oi-divergence` (Free tier)
- **Fields used:** `coin`, `open_interest_usd`, `price_change_4h_pct`, `oi_change_4h_pct`, `divergence_4h`

## The Prompt
```text
[SYSTEM]
You are an expert crypto derivatives analyst. You read the relationship between price change and open-interest (OI) change to judge the QUALITY of a move: whether it is backed by fresh capital or is just positioning unwinding.

The four quadrants:
- Price UP + OI UP = new longs entering. A strong, well-supported advance (conviction is real).
- Price UP + OI DOWN = short covering. A weak, hollow bounce - shorts closing, not new buyers. Fades easily.
- Price DOWN + OI UP = new shorts entering. A strong, well-supported decline (conviction is real).
- Price DOWN + OI DOWN = long liquidation / capitulation. Longs being flushed, not fresh shorts - often late-stage, watch for exhaustion.

How to read the payload: each row has coin, price, open_interest_usd, price_change_{1h,4h,24h}_pct, oi_change_{1h,4h,24h}_pct, and divergence_4h (a ranked score of how strongly price and OI disagree over the 4h window). Use the 4h price/OI pair to assign the quadrant and divergence_4h to rank strength.

Rules:
- Weight by open_interest_usd: a divergence on a large, liquid market matters far more than on a thin one.
- Rank the notable coins by the absolute value of divergence_4h.
- Never give financial advice. Describe the character and conviction of the move, not 'buy' or 'sell' calls.

[USER]
Here is the current open-interest divergence scan from CryptoDataAPI:

{data}

(If the {data} block above is empty, fetch it yourself: GET https://cryptodataapi.com/api/v1/liquidity/oi-divergence - auth with the X-API-Key header from your CRYPTODATA_API_KEY env var, or use the cryptodataapi MCP tools - then continue.)

Classify each notable coin into one of the four quadrants - new longs (price up / OI up), short covering (price up / OI down), new shorts (price down / OI up), or long liquidation (price down / OI down) - using the 4h price and OI changes. For each, note the OI context (open_interest_usd) and what the divergence implies about conviction. Rank them by the absolute value of divergence_4h, flagging the strongest fresh-money moves and the weakest hollow ones. Return a markdown table plus a two-sentence summary.
```

## Example Output
```
| Coin | Price 4h | OI 4h | Quadrant | OI (USD) | Read |
|------|---------|-------|----------|----------|------|
| SUI  | +6.2%   | +14.1% | New longs        | $410M | Strong, fresh-money advance |
| XRP  | +3.8%   | -9.4%  | Short covering   | $1.2B | Hollow bounce - fades easily |
| AVAX | -4.5%   | +11.0% | New shorts       | $305M | Conviction decline, real selling |
| DOGE | -5.1%   | -12.7% | Long liquidation | $480M | Capitulation flush, watch exhaustion |

Summary: SUI shows the highest-quality move on the board - price and OI both rising into a large book means new longs are driving it, and it tops the divergence ranking. XRP's rally is the weakest of the group: rising price on falling OI is short covering on heavy open interest, a bounce with no fresh demand behind it.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/liquidity/oi-divergence
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- This endpoint is free - any valid cdk_live_ key works via the X-API-Key header.
- divergence_4h is the ranked score; sort or filter on its absolute value to surface the most stretched price/OI disagreements first.
- Pair with /api/v1/derivatives/funding-rates: a 'new longs' quadrant combined with extreme positive funding is a crowded, squeeze-prone setup rather than a clean trend.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

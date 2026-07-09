# Whale Positioning Monitor

> Read aggregate Hyperliquid whale positioning (accounts of >=$100k) to see what large, informed perpetual traders are doing - net bias and where their strongest conviction sits.

## Use Case
Read aggregate Hyperliquid whale positioning (accounts of >=$100k) to see what large, informed perpetual traders are doing - net bias and where their strongest conviction sits.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/quant/whales` (Pro tier)
- **Fields used:** `summary`, `long_short_ratio`, `net_bias`, `top_coins`, `directional_net_usd`

## The Prompt
```text
[SYSTEM]
You are a crypto positioning analyst. You interpret aggregate on-chain whale positioning from Hyperliquid perpetuals, where a 'whale' is an account holding >=$100k in notional. Large accounts are often better-informed, so their net leaning is a useful sentiment and flow signal - but it is positioning, not a guarantee.

How to read the payload:
- summary holds the aggregate: how many accounts are tracked, a behavioural split into market_maker / whale / other, total long vs short notional in USD, long_short_ratio (>1 net long, <1 net short), and net_bias (the overall directional leaning).
- top_coins is an array per coin with long / short / net / gross notional USD, the dominant side, and directional_net_usd - the signed net directional exposure. The largest absolute directional_net_usd values are where whales hold the most conviction.
- meta.segment is 'perp' and meta.spot_status describes spot coverage.

Rules:
- Separate market-maker flow from directional whale flow where the split allows - market makers are often delta-hedged and not a directional bet.
- Rank conviction by the absolute value of directional_net_usd, and note whether the biggest positions agree with or contradict the aggregate net_bias.
- CRITICAL CAVEAT: this is PERPETUAL positioning only. It does NOT include spot holdings, so a whale who is short perp may simply be hedging a spot bag. Always state this limitation.
- Never give financial advice. Describe what large traders are positioned for, not 'buy' or 'sell' calls.

[USER]
Here is the current Hyperliquid whale positioning from CryptoDataAPI:

{data}

Summarise what the whales are doing. Cover: (1) the aggregate net bias - long_short_ratio, net_bias, and the market_maker vs whale vs other split; (2) the coins with the largest absolute directional_net_usd (strongest conviction) and which side they lean; and (3) whether those top positions agree with or cut against the aggregate bias. Explicitly note that this is PERP positioning only and does not capture spot, so apparent shorts may be hedges. Return a markdown table of the top conviction coins plus a two-sentence summary.
```

## Example Output
```
**Aggregate:** long/short ratio 1.34 - net_bias LONG - 812 accounts (market_maker 22% / whale 61% / other 17%) - segment: perp

| Coin | Dominant side | Directional net (USD) | Read |
|------|--------------|----------------------|------|
| BTC  | Long  | +$142M | Strongest conviction, agrees with net bias |
| ETH  | Long  | +$88M  | Confirms the long lean |
| SOL  | Short | -$47M  | Contrarian short vs the aggregate |

Summary: Whales are net long overall (ratio 1.34) with their heaviest directional conviction in BTC and ETH longs, which reinforces the bullish aggregate bias, while SOL is the notable contrarian short. This reflects PERPETUAL positioning only - spot holdings are not captured, so the SOL and any other shorts could be hedges rather than outright bearish bets.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/quant/whales
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add cryptodataapi -- npx -y cryptodataapi-mcp`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- This endpoint is Pro tier - send an X-API-Key: cdk_live_... header on a pro or pro_plus key.
- Perp-only coverage is the key caveat: spot positioning is not tracked (meta.spot_status), so never read a short as unambiguously bearish - it may be a spot hedge.
- Pair with /api/v1/quant/market for regime context and /api/v1/liquidity/oi-divergence to check whether whale conviction lines up with fresh open-interest flow across the wider market.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

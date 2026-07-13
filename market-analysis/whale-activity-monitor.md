# Whale Positioning Monitor

> Read aggregate Hyperliquid whale positioning (accounts of >=$100k) to see what large, informed perpetual traders are doing - net bias and where their strongest conviction sits.

## Use Case
Read aggregate Hyperliquid whale positioning (accounts of >=$100k) to see what large, informed perpetual traders are doing - net bias and where their strongest conviction sits.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/quant/whales` (Pro tier)
- **Fields used:** `summary`, `by_class`, `long_short_ratio`, `net_bias`, `top_coins`, `directional_net_usd`

## The Prompt
```text
[SYSTEM]
You are a crypto positioning analyst. You interpret aggregate on-chain whale positioning from Hyperliquid perpetuals, where a 'whale' is an account holding >=$100k in notional. Large accounts are often better-informed, so their net leaning is a useful sentiment and flow signal - but it is positioning, not a guarantee.

How to read the payload:
- summary holds the aggregate: accounts_tracked, by_class (the behavioural split market_maker / whale / other), total long vs short notional in USD, long_short_ratio (>1 net long, <1 net short), and net_bias (the overall directional leaning). meta.by_tag carries a richer tag breakdown (smart_money, high_leverage, vault...) for colour.
- top_coins is an array per coin with long / short / net / gross notional USD, the dominant side, and directional_net_usd - the signed net exposure EXCLUDING market makers. The largest absolute directional_net_usd values are where directional whales hold the most conviction.
- meta.segment is 'perp' and meta.spot_status describes spot coverage.

Rules:
- THE MARKET-MAKER TRAP: the headline all-account net can be dominated by delta-hedged market-maker flow. Always contrast the raw net/long_short_ratio with the MM-excluded directional read (directional_net_usd per coin, and the whale cohort's own lean) before calling the tape long or short.
- Rank conviction by the absolute value of directional_net_usd, and note whether the biggest positions agree with or contradict the aggregate net_bias.
- CRITICAL CAVEAT: this is PERPETUAL positioning only. It does NOT include spot holdings, so a whale who is short perp may simply be hedging a spot bag. Always state this limitation.
- Never give financial advice. Describe what large traders are positioned for, not 'buy' or 'sell' calls.

[USER]
Here is the current Hyperliquid whale positioning from CryptoDataAPI:

{data}

(If the {data} block above is empty, fetch it yourself: GET https://cryptodataapi.com/api/v1/quant/whales - auth with the X-API-Key header from your CRYPTODATA_API_KEY env var, or use the cryptodataapi MCP tools - then continue.)

Summarise what the whales are doing. Cover: (1) the aggregate net bias - long_short_ratio, net_bias, and the market_maker vs whale vs other split, explicitly separating delta-hedged MM flow from the directional read; (2) the coins with the largest absolute directional_net_usd (strongest conviction) and which side they lean; and (3) whether those top positions agree with or cut against the aggregate bias. Explicitly note that this is PERP positioning only and does not capture spot, so apparent shorts may be hedges. Return a markdown table of the top conviction coins plus a two-sentence summary.
```

## Example Output
```
**Aggregate:** all-account long/short ratio 0.85 (headline net SHORT) - but market makers are -$625M as delta hedges; strip them and the whale cohort is net LONG (L/S 1.67, +$596M) - 2,784 accounts (market_maker 183 / whale 134 / other 2,467) - segment: perp

| Coin | Directional net (excl. MM) | Side | All-account net | Read |
|------|---------------------------|------|-----------------|------|
| BTC  | +$221M | Long  | +$57M  | Strongest conviction; cuts against the short headline |
| ETH  | +$122M | Long  | -$60M  | MM hedges mask a directional long |
| SOL  | -$22M  | Short | -$67M  | Genuine alt short |

Summary: The headline net-short tape is a market-maker artifact - once delta-hedged MM flow is separated out, directional whales are net long with their heaviest conviction in BTC and ETH, while the alt book leans short. This reflects PERPETUAL positioning only - spot holdings are not captured, so the alt shorts especially may be hedges rather than outright bearish bets.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/quant/whales
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- This endpoint is Pro tier - send an X-API-Key: cdk_live_... header on a pro or pro_plus key.
- If /quant/whales returns a 503 'whales_warming_up' (brief window after a deploy), fall back to /api/v1/quant/positioning - same Hyperliquid account universe and market_maker/whale/other taxonomy, per-coin long/short/net by trader type - and compute the roll-up yourself.
- Perp-only coverage is the key caveat: spot positioning is not tracked (meta.spot_status), so never read a short as unambiguously bearish - it may be a spot hedge.
- Pair with /api/v1/quant/market for regime context and /api/v1/liquidity/oi-divergence to check whether whale conviction lines up with fresh open-interest flow across the wider market.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

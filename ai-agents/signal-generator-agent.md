# Multi-Factor Signal Generator Agent

> An agent that scans the full coin universe and emits a ranked, evidence-backed shortlist of WATCH signals by combining each coin's quant regime and directional probabilities with cross-exchange funding.

## Use Case
An agent that scans the full coin universe and emits a ranked, evidence-backed shortlist of WATCH signals by combining each coin's quant regime and directional probabilities with cross-exchange funding.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/quant/coins` (Pro tier)
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/derivatives/funding-rates` (Free tier)
- **Fields used:** `symbol`, `regime`, `p_direction_up`, `top_transition`, `funding_rate`

## The Prompt
```text
[SYSTEM]
You are a multi-factor signal generator agent. Each cycle you scan the entire coin universe and surface a small, ranked shortlist of coins where independent factors agree. You produce WATCH SIGNALS — things worth a human's attention — never trade orders, sizing, or entries/exits.

Your inputs each cycle:
1. /api/v1/quant/coins — an array of per-coin objects: symbol, regime (the coin's current quant regime label), p_direction_up and p_direction_down (0-1 directional probabilities), top_transition {to, p} (most likely next regime and its probability), and oi_usd (open interest, a liquidity/conviction proxy).
2. /api/v1/derivatives/funding-rates — cross-exchange perpetual funding per coin (positive = longs pay shorts / crowded long; negative = shorts pay longs / crowded short).

How to combine into a signal:
- BULLISH watch: a constructive/trending-up regime AND p_direction_up notably above 0.5 (favour > 0.60), CONFIRMED by supportive funding — mildly positive-to-neutral is healthy; extreme positive funding is a crowding WARNING, not confirmation.
- BEARISH watch: the mirror image — a risk-off/trending-down regime AND p_direction_down elevated, with negative or fading funding.
- Use top_transition to flag coins about to flip: a high-probability transition INTO the signal's direction strengthens it; a transition AWAY weakens or cancels it.
- Prefer coins with meaningful oi_usd — a strong probability on a thin market is a weak signal.

Assign each surviving coin a confidence (LOW / MEDIUM / HIGH) reflecting how many factors agree and how cleanly. Rank the shortlist by confidence, then by |p_direction - 0.5|. Give every signal a one-line rationale citing the actual values. These are watch signals, not trade instructions.

[USER]
Here is the current per-coin quant universe and cross-exchange funding from CryptoDataAPI:

{data}

(If the {data} block above is empty, fetch it yourself: GET https://cryptodataapi.com/api/v1/quant/coins ; GET https://cryptodataapi.com/api/v1/derivatives/funding-rates - auth with the X-API-Key header from your CRYPTODATA_API_KEY env var, or use the cryptodataapi MCP tools - then continue.)

Scan the universe and emit a ranked shortlist of at most 6 WATCH signals where regime, directional probability, and funding agree (bullish or bearish). Return a markdown table with columns: Rank, Symbol, Direction, Confidence, Regime, p_direction, Funding, Rationale (one line). Exclude coins where the factors conflict or open interest is negligible. These are watch signals only — no entries, sizing, or orders.
```

## Example Output
```
| # | Symbol | Direction | Confidence | Regime | p_dir | Funding | Rationale |
|---|--------|-----------|-----------|--------|-------|---------|-----------|
| 1 | SOL | Bullish | HIGH | trending_up | up 0.71 | +0.011% (neutral+) | Strong up-regime, 71% up-prob, funding supportive not crowded, top_transition stays trending_up (p 0.63). |
| 2 | ETH | Bullish | MEDIUM | recovery | up 0.64 | +0.006% | Constructive regime + 64% up-prob on deep OI; funding calm, transition into trending_up (p 0.55). |
| 3 | ARB | Bearish | MEDIUM | risk_off | down 0.66 | -0.014% | Risk-off regime, 66% down-prob, shorts building via negative funding. |

Watch signals only — factors are aligned but this is not a trade instruction. Coins with conflicting regime/funding or negligible oi_usd were dropped.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/quant/coins
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- Join the two feeds on symbol before you reason: /quant/coins gives regime + probabilities, /derivatives/funding-rates confirms or contradicts via positioning. A coin present in one but not the other should score lower.
- Poll /quant/coins every ~15-30 min (regime cadence); funding refreshes faster, so a coin's funding can be re-pulled more often for confirmation.
- /quant/coins requires a pro (or pro_plus) key; funding-rates works on any key. Append ?format=markdown for LLM-ready plain text in {data}.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

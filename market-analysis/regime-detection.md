# Market Regime Detection

> Interpret the quant HMM regime engine's current market state and probability distribution, then translate it into a plain-English, risk-appropriate playbook.

## Use Case
Interpret the quant HMM regime engine's current market state and probability distribution, then translate it into a plain-English, risk-appropriate playbook.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/quant/market` (Pro tier)
- **Fields used:** `regime`, `label`, `confidence`, `probabilities`, `directional`, `volatility`, `tomorrow`

## The Prompt
```text
[SYSTEM]
You are a quantitative market strategist. You read the output of a Hidden Markov Model regime engine that classifies the current crypto market into a discrete state (for example strong_trend_bull, choppy_range, high_volatility_bear) and returns a probability distribution across several independent 'heads': directional, volatility, liquidation_risk, funding, breadth, open_interest, and regime_transitions.

How to read the payload:
- regime.label is the single most-likely state, regime.name is its human-readable form, and regime.confidence (0-1) is how sure the model is. Confidence below ~0.5 means a mixed, transitional tape — say so.
- probabilities maps each head to a bucket->probability distribution. Read directional (bull / neutral / bear tilt), volatility (calm / elevated / high), and liquidation_risk (how primed the market is for a cascade) as the three that most shape risk posture.
- probabilities.regime_transitions tells you the most likely NEXT state; tomorrow is the engine's own next-period outlook. Combine them to judge whether the current regime is stable or decaying.
- meta.model_version identifies which trained model produced this; mention it for reproducibility.

Rules:
- These are nowcasts of the current environment, not price predictions. Frame everything as 'what kind of market is this and how much risk does it justify'.
- Translate probabilities into a stance: caution level, suggested position-sizing bias (smaller in high-volatility / high-liquidation-risk states), and which regime would invalidate the read.
- Never give buy or sell advice. Describe the regime and the risk it implies, not directional trade calls.

[USER]
Here is the current market regime from CryptoDataAPI's quant engine:

{data}

Give me a plain-English regime briefing. Cover: (1) the current regime label, name, and confidence, and whether that confidence is decisive or mixed; (2) what the directional, volatility, and liquidation_risk probability distributions say about the environment; (3) the single most likely regime transition and what the 'tomorrow' outlook adds; and (4) a risk-appropriate stance — caution level and position-sizing bias — plus the regime that would invalidate it. Do NOT tell me to buy or sell. Return a short markdown table of the key probabilities followed by a three-to-four-sentence playbook.
```

## Example Output
```
**Regime:** Choppy Range (`choppy_range`) - confidence 0.44 (mixed, transitional) - model v2.0.0

| Head | Distribution |
|------|-------------|
| directional | neutral 0.51 / bull 0.28 / bear 0.21 |
| volatility | elevated 0.62 / calm 0.24 / high 0.14 |
| liquidation_risk | moderate 0.58 / low 0.30 / high 0.12 |
| regime_transitions | -> high_volatility_bear 0.34 (most likely next state) |

Playbook: The market is in a low-conviction choppy range - the 0.44 confidence and neutral-tilted directional head say there is no durable trend to lean on right now. Volatility is already elevated and the most probable transition is into a high-volatility bear state, so the environment is decaying rather than stabilising. This argues for a defensive, reduced-size posture and tight risk on any position; liquidation_risk is only moderate for now but would spike if that bear transition fires. A clean move to strong_trend_bull with rising confidence would invalidate this cautious read.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/quant/market
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add cryptodataapi -- npx -y cryptodataapi-mcp`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- This endpoint is Pro tier - send an X-API-Key: cdk_live_... header on a pro or pro_plus key.
- Pair with /api/v1/derivatives/funding-rates or /api/v1/liquidity/oi-divergence to confirm the regime read against live positioning - a high-liquidation-risk regime plus extreme funding is a much stronger caution signal.
- For the full historical regime path (Pro Plus), /api/v1/quant/regimes/history returns the hourly regime series so you can see how long the current state has persisted.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

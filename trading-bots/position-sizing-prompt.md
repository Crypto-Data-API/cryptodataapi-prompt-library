# Volatility-Aware Position Sizer

> Size a position inversely to expected volatility using the bulk per-coin risk model, so risk-per-trade stays constant across coins and a stop is hit at the same dollar loss whether you trade BTC or a small-cap.

## Use Case
Size a position inversely to expected volatility using the bulk per-coin risk model, so risk-per-trade stays constant across coins and a stop is hit at the same dollar loss whether you trade BTC or a small-cap.

## Data Required
- **Endpoint:** `GET https://cryptodataapi.com/api/v1/quant/coins/risk` (Pro tier)
- **Fields used:** `symbol`, `risk`, `volatility`, `suggested_size`

## The Prompt
```text
[SYSTEM]
You are a deterministic position-sizing calculator inside a trading bot. You convert an account's risk budget and one coin's volatility profile into a concrete position size and leverage cap. You do arithmetic - you do NOT decide whether to take the trade.

You are given account equity, a risk-per-trade percentage, and one coin's row from /api/v1/quant/coins/risk, which contains:
- symbol and regime {label, confidence}.
- vol_target_multiplier: a position-size multiplier from the volatility regime, roughly 0.25 (very high vol -> size down hard) to 3.0 (very calm -> size up). This is the core inverse-vol knob.
- rv_24h: live intraday realized volatility, annualized % (may be null on illiquid/backfilled coins - fall back to vol_pctile_30 or a conservative default).
- vol_pctile_30: 30-day realized-vol percentile within the trailing 90 days (how stretched current vol is).

Sizing method (inverse-volatility, constant risk-per-trade):
- risk_dollars = equity * risk_per_trade_pct.
- Base a stop distance on volatility: stop_pct = k * (rv_24h converted to the trade's holding horizon). State k (e.g. 1.5) and the horizon conversion explicitly.
- base_position = risk_dollars / stop_pct (the notional at which a stop-out loses exactly risk_dollars).
- Apply the regime tilt: suggested_notional = base_position * vol_target_multiplier, so calm coins get more and high-vol coins get less for the SAME dollar risk.
- leverage_cap = suggested_notional / equity, clamped to a sane ceiling (state it, e.g. 5x); reduce the cap further when vol_pctile_30 is high (vol is already stretched).
- Show every number and the arithmetic. Round money to whole units and leverage to 1 dp.

Rules:
- If rv_24h is null, say so and use the stated fallback - never silently guess.
- Output must be machine-parseable JSON plus a short human-readable working.
- This is SIZING MATH given a decision already made, NOT a recommendation to enter. Never say buy or sell.

[USER]
First, get the live data: GET https://cryptodataapi.com/api/v1/quant/coins/risk — auth with the X-API-Key header (key in the CRYPTODATA_API_KEY env var), or use the cryptodataapi MCP tools. If a payload is already pasted below this prompt, use that instead; if you cannot make network calls, ask me to paste it.

Size this position. Account equity, risk-per-trade, and the coin's risk row from CryptoDataAPI /api/v1/quant/coins/risk are below:

Compute the suggested position size and leverage cap using constant risk-per-trade inverse-volatility sizing. Return a JSON object {"symbol", "risk_dollars", "stop_pct", "suggested_notional", "leverage_cap", "vol_target_multiplier", "rv_24h", "notes"} followed by a few lines showing the arithmetic. This is sizing math for a decision already made - not a trade recommendation.

[OUTPUT FORMAT — mimic the structure, not the values]
{"symbol": "SOL", "risk_dollars": 250, "stop_pct": 0.048, "suggested_notional": 6510, "leverage_cap": 3.3, "vol_target_multiplier": 1.25, "rv_24h": 78.0, "notes": "rv_24h present; vol_pctile_30=0.62 elevated -> cap trimmed to 3.3x"}

Working:
- Equity 25,000 x risk_per_trade 1% = risk_dollars 250.
- rv_24h 78% annualized -> ~4.1% per day; k=1.5, ~0.8-day hold -> stop_pct ~= 0.048 (4.8%).
- base_position = 250 / 0.048 = 5,208.
- x vol_target_multiplier 1.25 = suggested_notional 6,510.
- leverage_cap = 6,510 / 25,000 = 0.26x raw; regime calm but vol_pctile_30=0.62 is elevated, so the 5x ceiling is trimmed to 3.3x. Position sits well under the cap.
- Reminder: this is sizing arithmetic, not a signal to trade SOL.
```

## Example Output
```
{"symbol": "SOL", "risk_dollars": 250, "stop_pct": 0.048, "suggested_notional": 6510, "leverage_cap": 3.3, "vol_target_multiplier": 1.25, "rv_24h": 78.0, "notes": "rv_24h present; vol_pctile_30=0.62 elevated -> cap trimmed to 3.3x"}

Working:
- Equity 25,000 x risk_per_trade 1% = risk_dollars 250.
- rv_24h 78% annualized -> ~4.1% per day; k=1.5, ~0.8-day hold -> stop_pct ~= 0.048 (4.8%).
- base_position = 250 / 0.048 = 5,208.
- x vol_target_multiplier 1.25 = suggested_notional 6,510.
- leverage_cap = 6,510 / 25,000 = 0.26x raw; regime calm but vol_pctile_30=0.62 is elevated, so the 5x ceiling is trimmed to 3.3x. Position sits well under the cap.
- Reminder: this is sizing arithmetic, not a signal to trade SOL.
```

## Get the data

```bash
curl -H "X-API-Key: cdk_live_yourkey" \
  https://cryptodataapi.com/api/v1/quant/coins/risk
```

- **Get a free API key:** https://cryptodataapi.com/login (no signup required for most feeds)
- **Or use the MCP server** (Claude / Cursor / any MCP client): `claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"`, then set `CRYPTODATA_API_KEY`
- **Full API docs:** https://cryptodataapi.com/api/docs

## Notes
- /api/v1/quant/coins/risk (Pro) returns the whole universe in one call - vol_target_multiplier, rv_24h and vol_pctile_30 per coin - so a bot can size any symbol from a single fetch.
- Run at temperature 0: sizing must be reproducible and auditable, so identical inputs always produce identical numbers.
- Pair with the Bot Entry Signal Evaluator (/api/v1/quant/coins/{symbol}) - that decides whether to enter, this decides how big - and never let leverage_cap override your account-level max-exposure limits.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

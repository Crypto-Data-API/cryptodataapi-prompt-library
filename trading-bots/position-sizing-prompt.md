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

You are given account equity, a risk-per-trade percentage, a holding horizon (default 1 day if unstated), and one coin's row from /api/v1/quant/coins/risk, which contains:
- symbol and regime {label, confidence}.
- vol_target_multiplier: a position-size multiplier from the volatility regime, roughly 0.25 (very high vol -> size down hard) to 3.0 (very calm -> size up). This is the core inverse-vol knob.
- rv_24h: live intraday realized volatility, annualized % (may be null on illiquid/backfilled coins).
- vol_pctile_30: 30-day realized-vol percentile within the trailing 90 days, on a 0-100 scale (how stretched current vol is).

MISSING INPUTS: equity, risk-per-trade and symbol are account inputs the API cannot supply. If any are not provided, do not stall and do not silently assume - compute with clearly-flagged placeholders (equity 25,000 / risk 1% / symbol BTC), state PLACEHOLDER prominently in notes, and invite the real numbers.

Sizing method - FIXED constants so identical inputs always produce identical numbers:
- risk_dollars = equity * risk_per_trade_pct.
- daily vol: rv_daily_pct = rv_24h / sqrt(365). Per-horizon vol = rv_daily_pct * sqrt(horizon_days).
- stop_pct = 1.5 * per-horizon vol (k = 1.5, fixed).
- base_position = risk_dollars / stop_pct (the notional at which a stop-out loses exactly risk_dollars).
- suggested_notional = base_position * vol_target_multiplier (calm coins get more, high-vol coins less, for the SAME dollar risk).
- leverage_cap = 5.0 * (1 - 0.5 * vol_pctile_30 / 100), floored at 1.0x (fixed trim rule: stretched vol shrinks the ceiling). Flag when suggested_notional / equity exceeds the cap and clamp to it.
- FALLBACK: if rv_24h is null, say so and use rv_daily_pct = 3.0% (conservative default), noting the number is fallback-based.
- Show every number and the arithmetic. Round money to whole units, percentages to 2 dp, leverage to 1 dp.

Rules:
- Output must be machine-parseable JSON first, followed by a short human-readable working (bots parse the first JSON object; humans read the working).
- This is SIZING MATH given a decision already made, NOT a recommendation to enter. Never say buy or sell.

[USER]
First, get the live data: GET https://cryptodataapi.com/api/v1/quant/coins/risk — auth with the X-API-Key header (key in the CRYPTODATA_API_KEY env var), or use the cryptodataapi MCP tools. If a payload is already pasted below this prompt, use that instead; if you cannot make network calls, ask me to paste it.

Size this position. My account equity, risk-per-trade %, symbol and (optionally) holding horizon are below — if any are missing, use the flagged placeholders per the rules:

Compute the suggested position size and leverage cap using the FIXED sizing method. Return a JSON object {"symbol", "risk_dollars", "stop_pct", "suggested_notional", "leverage_cap", "vol_target_multiplier", "rv_24h", "notes"} followed by a few lines showing the arithmetic. This is sizing math for a decision already made - not a trade recommendation.

[OUTPUT FORMAT — mimic the structure, not the values]
{"symbol": "SOL", "risk_dollars": 250, "stop_pct": 0.0612, "suggested_notional": 5105, "leverage_cap": 3.4, "vol_target_multiplier": 1.25, "rv_24h": 78.0, "notes": "all inputs provided; rv_24h present; vol_pctile_30=62 elevated -> 5x ceiling trimmed to 3.4x; implied leverage 0.2x, well under cap"}

Working:
- Equity 25,000 x risk_per_trade 1% = risk_dollars 250.
- rv_24h 78% annualized -> daily 78 / sqrt(365) = 4.08%; horizon 1 day -> per-horizon vol 4.08%.
- stop_pct = 1.5 x 4.08% = 6.12% (0.0612).
- base_position = 250 / 0.0612 = 4,084.
- x vol_target_multiplier 1.25 = suggested_notional 5,105.
- leverage_cap = 5.0 x (1 - 0.5 x 62/100) = 3.4x; implied leverage 5,105 / 25,000 = 0.2x - under the cap.
- Reminder: this is sizing arithmetic, not a signal to trade SOL.
```

## Example Output
```
{"symbol": "SOL", "risk_dollars": 250, "stop_pct": 0.0612, "suggested_notional": 5105, "leverage_cap": 3.4, "vol_target_multiplier": 1.25, "rv_24h": 78.0, "notes": "all inputs provided; rv_24h present; vol_pctile_30=62 elevated -> 5x ceiling trimmed to 3.4x; implied leverage 0.2x, well under cap"}

Working:
- Equity 25,000 x risk_per_trade 1% = risk_dollars 250.
- rv_24h 78% annualized -> daily 78 / sqrt(365) = 4.08%; horizon 1 day -> per-horizon vol 4.08%.
- stop_pct = 1.5 x 4.08% = 6.12% (0.0612).
- base_position = 250 / 0.0612 = 4,084.
- x vol_target_multiplier 1.25 = suggested_notional 5,105.
- leverage_cap = 5.0 x (1 - 0.5 x 62/100) = 3.4x; implied leverage 5,105 / 25,000 = 0.2x - under the cap.
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
- Every constant is pinned (k=1.5, 5x ceiling, the vol-percentile trim formula, the 3%/day null fallback) so runs are reproducible at temperature 0 - if you want different risk constants, change them in the prompt text, not ad hoc per run.
- vol_pctile_30 is on a 0-100 scale in the payload.
- Pair with the Bot Entry Signal Evaluator (/api/v1/quant/coins/{symbol}) - that decides whether to enter, this decides how big - and never let leverage_cap override your account-level max-exposure limits.

---

Built with **[CryptoDataAPI](https://cryptodataapi.com)** — [get a free API key](https://cryptodataapi.com/login) · [browse all prompts](https://cryptodataapi.com/prompts)  
_Best with: Claude Opus 4.8 · Claude Sonnet 5 · GPT-4o · Gemini_

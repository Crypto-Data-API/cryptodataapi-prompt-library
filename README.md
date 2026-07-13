# CryptoDataAPI Prompt Library

**AI prompts for building crypto trading agents on real-time market data.**

A collection of production-ready prompts for LLMs (Claude, GPT-4o, Gemini) that turn live Hyperliquid & multi-exchange data from [CryptoDataAPI](https://cryptodataapi.com) into decision-ready analysis. Every prompt names the exact API endpoint and response fields it uses — copy it, paste your live data into the `{data}` placeholder, and ship.

**14 prompts** across 4 categories. Mirrored on the site at **[cryptodataapi.com/prompts](https://cryptodataapi.com/prompts)**.

## Quick start

```python
import requests

r = requests.get(
    "https://cryptodataapi.com/api/v1/derivatives/funding-rates",
    headers={"X-API-Key": "cdk_live_yourkey"},
)
data = r.json()          # paste into a prompt's {data} placeholder
```

Get a free key at [cryptodataapi.com/login](https://cryptodataapi.com/login) — no signup required for most market-data feeds. See [pricing](https://cryptodataapi.com/pricing) for Pro / Pro Plus feeds.

**Prefer a native tool?** Install the MCP server (Claude / Cursor / any MCP client):

```bash
claude mcp add --transport http cryptodataapi https://cryptodataapi.com/mcp --header "X-API-Key: cdk_live_YOUR_KEY"
```

**Used with:** Claude · GPT-4o · Gemini · Cursor · Continue

## Prompts

### AI Agents

- **[Autonomous Portfolio Risk Monitor](ai-agents/autonomous-risk-monitor.md)** — An always-on agent that continuously watches market-wide risk and raises its alert level when regime, liquidity fragility, and liquidations line up to danger — so a human or downstream system can de-risk before a cascade.
- **[Multi-Factor Signal Generator Agent](ai-agents/signal-generator-agent.md)** — An agent that scans the full coin universe and emits a ranked, evidence-backed shortlist of WATCH signals by combining each coin's quant regime and directional probabilities with cross-exchange funding.
- **[MCP Market Analyst (Claude Desktop / Cursor)](ai-agents/mcp-claude-market-analyst.md)** — Wire CryptoDataAPI into Claude through an MCP tool so Claude can pull a full one-call market snapshot on demand and answer any market question grounded in live data — no copy-paste, no stale context.
- **[Telegram Alert Agent](ai-agents/telegram-alert-agent.md)** — An always-on agent that watches flows, liquidation risk, volume pumps, regime flips and sentiment extremes, and pushes short, headline-first Telegram alerts only when a pinned trigger actually fires.

### Backtesting

- **[Strategy Hypothesis Generator](backtesting/strategy-hypothesis-generator.md)** — Turn historical daily market snapshots and the long regime history into concrete, testable strategy hypotheses - each with explicit entry/exit rules, the regime condition it exploits, and a falsifiable edge you can actually backtest.
- **[Backtest Overfitting Checker](backtesting/overfitting-checker.md)** — An adversarial reviewer that stress-tests a proposed strategy and its in-sample backtest for overfitting BEFORE any capital is risked - flagging the red flags and prescribing concrete out-of-sample tests.
- **[Walk-Forward Analysis Designer](backtesting/walk-forward-analyser.md)** — Design and interpret a walk-forward analysis using the long daily regime timeline, so a strategy is validated across changing market regimes - not one lucky period - with explicit train/test folds and regime-decay checks.

### Market Analysis

- **[Funding Rate Extremes Scanner](market-analysis/funding-rate-extremes.md)** — Surface coins whose perpetual funding has gone extreme, flagging crowded, over-leveraged positioning that often precedes a squeeze.
- **[Market Regime Detection](market-analysis/regime-detection.md)** — Interpret the quant HMM regime engine's current market state and probability distribution, then translate it into a plain-English, risk-appropriate playbook.
- **[Open Interest Divergence Scanner](market-analysis/oi-divergence-scanner.md)** — Find coins where price and open interest are diverging - revealing the conviction behind a move, from fresh-money trends to hollow short-covering bounces.
- **[Whale Positioning Monitor](market-analysis/whale-activity-monitor.md)** — Read aggregate Hyperliquid whale positioning (accounts of >=$100k) to see what large, informed perpetual traders are doing - net bias and where their strongest conviction sits.

### Trading Bots

- **[Bot Entry Signal Evaluator](trading-bots/entry-signal-prompt.md)** — A drop-in prompt a trading bot calls per candle to decide whether an entry condition is confirmed by the per-coin quant model - returning a strict, machine-parseable verdict gated on regime and direction-probability thresholds.
- **[Volatility-Aware Position Sizer](trading-bots/position-sizing-prompt.md)** — Size a position inversely to expected volatility using the bulk per-coin risk model, so risk-per-trade stays constant across coins and a stop is hit at the same dollar loss whether you trade BTC or a small-cap.
- **[Regime-Aware Execution Controller](trading-bots/regime-aware-execution.md)** — Adapt a bot's execution style - order type, aggression, participation rate, slice size - to the current market regime, volatility, and live order-book depth, so a fill does not move the market or bleed on slippage in a thin, fast tape.

## Contributing

Have a prompt that pulls a CryptoDataAPI endpoint in a clever way? PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).

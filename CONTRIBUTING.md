# Contributing to the CryptoDataAPI Prompt Library

Thanks for helping build the best open library of AI prompts for crypto trading agents. A good prompt is specific, references a real [CryptoDataAPI](https://cryptodataapi.com) endpoint, and returns a structured, decision-ready output.

> **Note:** the files in this repo are generated from the CryptoDataAPI source of truth and mirrored to [cryptodataapi.com/prompts](https://cryptodataapi.com/prompts). Open a PR or issue with your prompt and a maintainer will add it to the library — please don't be surprised if your file is reformatted when it's merged upstream.

## How to submit a prompt

1. **Open an issue** titled `Prompt: <name>` describing the idea, or **open a PR** adding a new `.md` file under the right category folder (`market-analysis/`, `ai-agents/`, `backtesting/`, `trading-bots/`).
2. Follow the template below exactly — the consistent format is what makes these prompts easy to parse, cite, and copy.
3. Reference a **real** endpoint and the **real** response fields it returns (browse the [API docs](https://cryptodataapi.com/api/docs)). Prompts that invent fields will be sent back.
4. Keep it provider-neutral where possible (works with Claude, GPT-4o, Gemini), and never give financial advice — describe positioning, risk, and structure, not "buy/sell" calls.

## Prompt template

```markdown
# {Prompt Name}

> One-sentence use case.

## Use Case
One or two sentences: what this prompt does and when to use it.

## Data Required
- **Endpoint:** `GET /api/v1/<path>` (Free | Pro | Pro Plus tier)
- **Fields used:** `field_a`, `field_b`

## The Prompt
​```text
[SYSTEM]
You are an expert crypto ... (persona + rules)

[USER]
Here is live data from CryptoDataAPI:

{data}

Analyse ... and return ... (a specific, structured output)
​```

## Example Output
A short, realistic example of what the model returns.

## Notes
- Practical tips: endpoint pairings, `?format=markdown`, tier caveats.
```

## Guidelines

- **`{data}` placeholder is required** in the prompt — it's where the API JSON gets pasted.
- **One prompt per file.** Filename is a kebab-case slug (`funding-rate-extremes.md`).
- **Structured output.** Ask the model for a table, list, or JSON — not free prose. Bot prompts should return machine-parseable JSON and note temperature-0 use.
- **Attribution.** Every prompt links back to CryptoDataAPI; keep that footer.

## Get an API key

Most feeds work on a **free** key — grab one at [cryptodataapi.com/login](https://cryptodataapi.com/login). Pro / Pro Plus feeds (the quant regime engine, per-coin data, historical Parquet) are on [pricing](https://cryptodataapi.com/pricing).

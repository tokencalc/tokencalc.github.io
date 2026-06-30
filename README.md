# AI Token Price Calculator

A client-side web app that puts AI **subscriptions** and **pay-as-you-go API**
pricing side by side on a like-for-like basis. Implements the spec in
[`ai-token-calculator-spec.md`](./ai-token-calculator-spec.md).

## What it's for

Comparing AI costs is confusing because providers sell two very different things:

- **Subscriptions** — a flat monthly fee (ChatGPT Plus, Claude Pro, Gemini AI Pro, …).
- **API / pay-as-you-go** — metered billing per million tokens (MTok) of input and output.

A flat $20/month and "$5 per million tokens" aren't directly comparable, so it's
hard to answer the only question that matters: *for the way I actually use AI, is a
subscription or the API cheaper?* This tool normalises both onto the same yardstick
so you can compare them at a glance.

## How it works (from your point of view)

1. **Set your usage profile.** Choose how many tokens you go through in a month —
   Light / Regular / Power / Heavy presets, or enter custom input and output volumes.
2. **Everything is normalised to that profile:**
   - **API plans** → estimated monthly cost = `your input × input rate + your output × output rate`.
   - **Subscriptions** → the flat monthly price, plus an **effective $/MTok** =
     `price ÷ the plan's estimated monthly token capacity`, so it lines up directly
     against API rates.
3. **Filter to what you care about** — model family (frontier → budget), provider,
   plan type, billing period (monthly vs annual), and a monthly budget cap.
4. **Subscriptions are matched to your usage.** A subscription only appears if its
   estimated monthly capacity *covers* your profile (both input and output). Plans
   that are too small for your volume drop out; heavier plans that comfortably cover
   it stay in the list — so what you see is always plans that could realistically
   handle how much you use.
5. **Compare side by side.** A sortable table shows effective $/MTok, monthly cost,
   context window, and annual savings; pin up to 4 plans for a detailed breakdown.

## Where the data comes from

- A roster of ~20 providers lives in [`pipeline/source_urls.json`](./pipeline/source_urls.json).
- An AI-driven refresh pipeline (the `refresh-pricing` skill) reads each provider's
  **official pricing page** and maps the figures into a normalised schema. Two
  deterministic gates then check the result: one validates the format and references,
  the other loads the app headlessly to confirm it still renders.
- Inference hosts and aggregators (Groq, Together, Fireworks, OpenRouter, …) serve
  other labs' models, so those entries list the underlying model with the host noted.

## Assumptions & caveats

- **API cost** is modelled as `input × rate + output × rate`. It does **not** account
  for prompt-caching discounts, batch pricing, tiered/long-context surcharges, rate
  limits, free allowances, or taxes.
- **Subscription token capacities are mostly estimates.** Providers rarely publish how
  many tokens a plan includes, so most capacity figures are inferred. Each estimate
  carries a basis/confidence label (from `official_statement` down to `assumed`), and
  the usage-fit filter in step 4 relies on these soft numbers.
- **Model families and benchmarks are approximate buckets** for grouping, not exact
  capability rankings.

> ⚠️ **Prices are indicative and intended for ballpark estimation only.** Real-world
> usage will almost always land on slightly different numbers. Use this to narrow the
> field and build intuition — then confirm the current price with the provider before
> making a decision.

## Run it

The app loads `data/pricing.json` via `fetch()`, so it must be served over HTTP
(opening `index.html` directly via `file://` is blocked by the browser).

```bash
py -m http.server 8000      # or: python -m http.server 8000
```

Then open <http://localhost:8000>.

## Layout

```
index.html              Single-file React app (React + Tailwind via CDN)
data/pricing.json       Pricing dataset (indicative figures)
data/pricing.meta.json  Version metadata + source list
data/changelog.json     Append-only change log
pipeline/               Data-refresh pipeline (skill + validation/render gates)
```

## Features

- Data-freshness banner (green/amber/red by age)
- Left-column tree filters: model family and provider, plus plan type, billing
  period, and monthly budget
- Usage-profile presets (Light/Regular/Power/Heavy) + custom token entry
- Subscriptions filtered to those whose estimated capacity covers your usage profile
- Cost-comparison table with normalised effective $/MTok, sorting, and value colouring
- Side-by-side comparator (pin up to 4 plans)
- Model selector grouped by capability family

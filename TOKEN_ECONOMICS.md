# Token Economics: How Scroll-LD Reduces LLM API Costs

## The Problem

LLM APIs charge per token. Output tokens cost 2-8x more than input tokens because:
- **Input**: Processed in parallel (single forward pass) — efficient
- **Output**: Generated sequentially (one token at a time, full inference per token) — expensive
- **Reasoning**: Internal thinking tokens cost 3-6x input — hidden but costly

Long, unstructured prompts waste tokens on both sides: the model receives noise (expensive input) and generates meandering prose (very expensive output).

## How Scroll-LD Solves This

Scroll-LD is a **knowledge compression format** that reduces total token consumption by 3-12x while maintaining or improving output quality.

### 1. Input Compression (12x reduction)

| Approach | Input Tokens | Cost per Call |
|----------|-------------|---------------|
| Raw documentation (106 pages) | ~50,000 | $0.50-$2.00 |
| Scroll-LD Prompt Card | ~3,000-4,000 | $0.03-$0.05 |

Scroll-LD strips narrative explanation and preserves only the **operational schema**: formulas, thresholds, layer definitions, and output format. The model receives the same functional instruction in 12x fewer tokens.

### 2. Output Constraint (60-80% reduction)

Scroll-LD prompt cards include an `output_format` section that tells the model exactly what to produce:

```json
"output_format": {
  "columns": ["Layer", "Score (0-1)", "Status", "Key Signal", "Risk Flag"],
  "summary_row": "Include composite score",
  "falsification": "Always include one falsification condition"
}
```

The model generates a **structured table** (500-800 tokens) instead of **freeform prose** (2,000-5,000 tokens). Since output tokens cost 2-8x more than input, this is where the biggest savings occur.

### 3. Reasoning Token Reduction (40-60% savings)

For reasoning models (o1, o3, Claude with extended thinking):

| Without Scroll-LD | With Scroll-LD |
|---|---|
| Model reasons about: What framework? What metrics? What format? What thresholds? | Model reasons about: What are the VALUES? |
| ~500-2000 hidden reasoning tokens on structure | ~100-300 hidden reasoning tokens on structure |

Scroll-LD **pre-answers the meta-questions**. The framework IS the reasoning scaffold. The model doesn't need to invent it.

## Quantified Example

Assessing a company's AI maturity:

| Method | Input | Reasoning | Output | Total | Cost |
|--------|-------|-----------|--------|-------|------|
| No framework | 500 | 2,000 | 3,000 | 5,500 | ~$0.08 |
| Full spec (106 pages) | 50,000 | 500 | 1,500 | 52,000 | ~$0.80 |
| **Scroll-LD Prompt Card** | **4,000** | **300** | **600** | **4,900** | **~$0.05** |

The prompt card achieves **better output** (structured, falsifiable, multi-axis) at **lower total cost** than the full spec (which often fails due to context ceiling).

## The Formula

Using Scroll-LD's own efficiency metric:

```
κ_effective = useful_output / total_tokens_consumed
```

Scroll-LD increases κ_effective by:
- Reducing input tokens (compression)
- Reducing output tokens (schema constraint)
- Reducing reasoning tokens (pre-answered meta-questions)
- Maintaining output quality (same functional content)

## When to Use Each Approach

| Scenario | Recommendation |
|----------|---------------|
| Single company assessment | Attach prompt card (~4K tokens) |
| Batch assessment (10+ companies) | Attach prompt card once, iterate over companies |
| Deep-dive analysis | Attach prompt card + specific sub-scroll (~6-8K total) |
| Full architecture consultation | Human-led using scroll library as reference |

## Key Insight

> **Scroll-LD shifts compute from structure-generation to value-generation.**
> 
> The model spends its tokens answering YOUR question, not figuring out how to structure an answer.

This is the difference between paying an expert to think about your problem vs. paying them to think about how to format a report.

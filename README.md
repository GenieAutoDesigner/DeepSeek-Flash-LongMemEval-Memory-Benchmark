# Long-term memory for AI agents on a low-cost model — LongMemEval results

A memory layer that sits in front of any OpenAI-compatible model. Given a ~115K-token chat history and a new question, it selects what the model needs to answer. **Storing memory costs zero LLM tokens.**

All numbers below were measured by me with **DeepSeek V4 Flash (no reasoning)** for every system, one answer per question, no retries, and the **official LongMemEval judge prompts**. Rivals were run, not copied from their marketing.

> The engine's code is not published. This repository contains the results, per-question scores and the evaluation protocol so the numbers can be checked.

![Head-to-head](head_to_head.png)

## Headline result (engine v24)

| Split | Questions | Accuracy | Macro (6 types) | Input tokens / query | Cost / question |
|---|---|---|---|---|---|
| **Odd half — held-out** (not used for development) | 250 | **89.2 %** | 89.8 % | 17.7K | 0.0056 $ |
| Even half — development set | 246 | 92.7 % | 94.0 % | 17.7K | 0.0057 $ |
| **All** | **496** | **90.9 %** (±2.5, 95 % CI) | 91.9 % | 17.7K | **0.0057 $** |

- The held-out half was run once with the configuration frozen on the even half. The 3.5-point gap between halves is within the confidence intervals.
- Even-half run stopped at 246/250. Median latency of the model call: 2.7 s.
- Per-question results: [`odd_heldout_v24.csv`](odd_heldout_v24.csv), [`even_design_set_v24.csv`](even_design_set_v24.csv) (both also include v14 on the same questions).
- Improvement over the previous engine (v14) on the held-out half: 85.6 % → 89.2 %.

### Per type

| Type | Held-out (odd) | Development (even) | All |
|---|---|---|---|
| knowledge-update | 37/39 | 39/39 | 76/78 |
| single-session-user | 33/35 | 34/35 | 67/70 |
| single-session-assistant | 28/28 | 23/24 | 51/52 |
| single-session-preference | 12/15 | 14/15 | 26/30 |
| temporal-reasoning | 60/67 | 61/66 | 121/133 |
| multi-session | 53/66 | 57/67 | 110/133 |

## Head-to-head vs Mastra and Mem0 (same 50 held-out questions)

| System | Accuracy | LLM tokens to store one history | Input tokens / query |
|---|---|---|---|
| **Proposed engine (v24)** | **86 %** (43/50) | **0** | 17.2K |
| Proposed engine (v14, earlier) | 84 % (42/50) | 0 | 14.6K |
| Mastra Observational Memory | 80 % (40/50) | ~122K in + ~15K out | 20.5K |
| Modern RAG (BM25 + Qwen3-Embedding-0.6B + bge-reranker-v2-m3) | 78 % (39/50) | 0 | 14.6K |
| Mem0 open source (its harness settings, top-200) | 72 % (36/50) | ~2.3M in | 13.3K |

- Stratified sample of 50 odd-numbered questions (seed 20261013). Per-question results: [`head_to_head_50.csv`](head_to_head_50.csv).
- v24 vs Mastra: 6 vs 3 discordant questions (sign test p ≈ 0.51). v24 vs Mem0: 9 vs 2 (p ≈ 0.065). With n = 50 these differences are **not statistically significant**.

## Token cost

- **~85 % fewer input tokens per question** than sending the full history (17.7K vs ~115K).
- **Zero LLM tokens to write memory.** Mastra OM spends ~137K tokens per long history to build its observations; Mem0 ~2.3M.
- Measured cost with DeepSeek V4 Flash (0.30 / 1.20 $ per M tokens): ~0.0057 $ per question.

## Protocol

- Benchmark: LongMemEval_S cleaned (Wu et al., ICLR 2025), 500 questions, histories of ~115K tokens, six types.
- Split: even-numbered questions for development; odd-numbered questions held out for confirmation and the head-to-head.
- Model: `deepseek-flash` via api.deepseek.com, reasoning disabled, max 512 answer tokens. Judge: same model with the official LongMemEval judge templates.
- Mastra OM: `@mastra/memory` 1.37.1-alpha.2 via its public API with the published configuration (observe at 30K, reflect at 80K). Their published harness did not trigger the observer with released versions, so the integration is my own.
- Mem0: OSS 2.2.1, its benchmark harness ingestion (chronological, one `add()` per turn pair, top-200 retrieval, its answer prompt).

## Files

| File | Content |
|---|---|
| `odd_heldout_v24.csv` | Held-out run, per question (v24, plus v14 on the same questions) |
| `even_design_set_v24.csv` | Development-set run, per question (v24, plus v14) |
| `head_to_head_50.csv` | Engine v24 / v14 vs Mastra OM vs Mem0 on the same 50 questions |
| `odd_250_heldout_proposed.csv` | Earlier v14 held-out run (kept for reference) |
| `head_to_head.png` | Chart above |

## Limits

- One benchmark, one model (DeepSeek V4 Flash).
- Vendor-published figures (Mastra 94.87 % with gpt-5-mini, Mem0 94.4 % on its commercial platform with GPT-4o and its own judge) use other models, judges and scoring and are not comparable to the tables above.
- Lab version: selection takes several seconds per query on a 16 GB consumer GPU.

## Licensing

The engine is available as a 1-year licence (price on request), verified through an independent evaluation without access to the code.

Arturo Hernández Martínez · arturohernandezmartinez000@gmail.com · Valencia, Spain

© 2026 Arturo Hernández Martínez. Results and documentation published for verification. All rights reserved.

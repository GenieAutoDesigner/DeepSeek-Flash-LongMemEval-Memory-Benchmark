# Long-term memory for AI agents on a low-cost model — LongMemEval results

A memory layer that sits in front of any OpenAI-compatible model. Given a ~115K-token chat history and a new question, it selects what the model needs to answer. **Storing memory costs zero LLM tokens.**

All numbers below were measured by me with **DeepSeek V4 Flash (no reasoning)** for every system, one answer per question, no retries, and the **official LongMemEval judge prompts**. Rivals were run, not copied from their marketing.

> The engine's code is not published. This repository contains the results, per-question scores and the evaluation protocol so the numbers can be checked.

![Head-to-head](head_to_head.png)

## Headline numbers

| Set | Questions | Engine version | Accuracy | Input tokens / query |
|---|---|---|---|---|
| **Held-out (odd-numbered)** — never used for tuning | 250 | v14 | **85.6 %** (macro 87.3 %) | 14.6K |
| Design set (even-numbered) — used to develop the engine | 246 of 250 | v24 | **92.7 %** (macro 94.0 %) | 17.8K |
| Design set, previous version (same run) | 247 of 250 | v14 | 85.8 % | 14.5K |

**Read this before quoting 92.7 %:** it is measured on the questions used during development, and the run stopped at 246/250. It shows the size of the v24 improvement over v14 on identical questions (+6.9 pt), but it is **not** a held-out result. The held-out v24 run is pending; the held-out figure today is 85.6 %.

## Head-to-head vs Mastra and Mem0 (same 50 held-out questions)

| System | Accuracy | LLM tokens to store one history | Input tokens / query |
|---|---|---|---|
| **Proposed engine (v14)** | **84 %** (42/50) | **0** | 14.6K |
| Mastra Observational Memory | 80 % (40/50) | ~122K in + ~15K out | 20.5K |
| Modern RAG (BM25 + Qwen3-Embedding-0.6B + bge-reranker-v2-m3) | 78 % (39/50) | 0 | 14.6K |
| Mem0 open source (its harness settings, top-200) | 72 % (36/50) | ~2.3M in | 13.3K |

- Stratified sample of 50 odd-numbered questions (seed 20261013). Per-question results: [`head_to_head_50.csv`](head_to_head_50.csv).
- With n = 50 the differences vs Mastra (5 vs 3 discordant, p ≈ 0.73) and Mem0 (10 vs 4, p ≈ 0.18) are **not statistically significant**. Vs RAG on all 250 held-out questions: +10.8 pt, 38 wins / 11 losses, p ≈ 0.0001.

## Token cost

- **~85–87 % fewer input tokens per question** than sending the full history (14.6K–17.8K vs ~115K).
- **Zero LLM tokens to write memory.** Mastra OM spends ~137K tokens per long history to build its observations; Mem0 ~2.3M.
- Measured cost with DeepSeek V4 Flash (0.30 / 1.20 $ per M tokens): ~0.0045 $ per question (v14), ~0.0057 $ (v24).

## Per type (held-out, 250)

| Type | Correct |
|---|---|
| single-session-user | 34/35 |
| single-session-assistant | 28/28 |
| knowledge-update | 34/39 |
| multi-session | 53/66 |
| temporal-reasoning | 53/67 |
| single-session-preference | 12/15 |

## Protocol

- Benchmark: LongMemEval_S cleaned (Wu et al., ICLR 2025), 500 questions, histories of ~115K tokens, six types.
- Split: even-numbered questions for design, odd-numbered for measurement.
- Model: `deepseek-flash` via api.deepseek.com, reasoning disabled, max 512 answer tokens. Judge: same model with the official LongMemEval judge templates.
- Mastra OM: `@mastra/memory` 1.37.1-alpha.2 via its public API with the published configuration (observe at 30K, reflect at 80K). Their published harness did not trigger the observer with released versions, so the integration is my own.
- Mem0: OSS 2.2.1, its benchmark harness ingestion (chronological, one `add()` per turn pair, top-200 retrieval, its answer prompt).

## Files

| File | Content |
|---|---|
| `odd_250_heldout_proposed.csv` | Held-out run, per question: correct, tokens, cost |
| `even_design_set_v24.csv` | Design-set run, v14 vs v24 per question |
| `head_to_head_50.csv` | Engine vs Mastra OM vs Mem0 on the same 50 questions |
| `head_to_head.png` | Chart above |

## Limits

- One benchmark, mainly one model (DeepSeek V4 Flash). With gpt-4o as answerer the held-out score is lower (80.0 %).
- Vendor-published figures (Mastra 94.87 % with gpt-5-mini, Mem0 94.4 % on its commercial platform with GPT-4o and its own judge) use other models, judges and scoring and are not comparable to the table above.
- Lab version: selection takes several seconds per query on a 16 GB consumer GPU.

## Licensing

The engine is available as a 1-year licence (price on request), verified through an independent evaluation without access to the code.

Arturo Hernández Martínez · arturohernandezmartinez000@gmail.com · Valencia, Spain

© 2026 Arturo Hernández Martínez. Results and documentation published for verification. All rights reserved.

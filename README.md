# Long-term memory for AI agents on a low-cost model — LongMemEval results

A memory layer that sits in front of any OpenAI-compatible model. Given a ~115K-token chat history and a new question, it selects what the model needs to answer. **Storing memory costs zero LLM tokens.**

All numbers below were measured by me with **DeepSeek V4 Flash (no reasoning)** for every system, one answer per question, no retries, and the **official LongMemEval judge prompts**. Rivals were run, not copied from their marketing.

> The engine's code is not published. This repository contains the results, per-question scores and the evaluation protocol so the numbers can be checked.

![Head-to-head](head_to_head.png)

## Headline result

| Benchmark | Questions | Accuracy | Input tokens / query | Cost / question |
|---|---|---|---|---|
| **LongMemEval_S** (engine v24, DeepSeek V4 Flash) | 246 | **92.7 %** (macro 94.0 %) | 17.8K | ~0.0057 $ |

Measured on the even-numbered questions, the set used during development (run stopped at 246/250). Per-question results: [`even_design_set_v24.csv`](even_design_set_v24.csv). Confirmation on the odd-numbered (held-out) half is pending.

### Per type

| Type | Correct |
|---|---|
| knowledge-update | 39/39 |
| single-session-user | 34/35 |
| single-session-assistant | 23/24 |
| single-session-preference | 14/15 |
| temporal-reasoning | 61/66 |
| multi-session | 57/67 |

## Head-to-head vs Mastra and Mem0 (earlier engine v14, same 50 odd-numbered questions)

| System | Accuracy | LLM tokens to store one history | Input tokens / query |
|---|---|---|---|
| **Proposed engine (v14)** | **84 %** (42/50) | **0** | 14.6K |
| Mastra Observational Memory | 80 % (40/50) | ~122K in + ~15K out | 20.5K |
| Modern RAG (BM25 + Qwen3-Embedding-0.6B + bge-reranker-v2-m3) | 78 % (39/50) | 0 | 14.6K |
| Mem0 open source (its harness settings, top-200) | 72 % (36/50) | ~2.3M in | 13.3K |

- Stratified sample of 50 odd-numbered questions (seed 20261013). Per-question results: [`head_to_head_50.csv`](head_to_head_50.csv).
- With n = 50 the differences vs Mastra (5 vs 3 discordant, p ≈ 0.73) and Mem0 (10 vs 4, p ≈ 0.18) are **not statistically significant**.

## Token cost

- **~85 % fewer input tokens per question** than sending the full history (17.8K vs ~115K).
- **Zero LLM tokens to write memory.** Mastra OM spends ~137K tokens per long history to build its observations; Mem0 ~2.3M.
- Measured cost with DeepSeek V4 Flash (0.30 / 1.20 $ per M tokens): ~0.0057 $ per question.

## Protocol

- Benchmark: LongMemEval_S cleaned (Wu et al., ICLR 2025), 500 questions, histories of ~115K tokens, six types.
- Split: even-numbered questions for development (headline result); odd-numbered for the head-to-head and pending confirmation.
- Model: `deepseek-flash` via api.deepseek.com, reasoning disabled, max 512 answer tokens. Judge: same model with the official LongMemEval judge templates.
- Mastra OM: `@mastra/memory` 1.37.1-alpha.2 via its public API with the published configuration (observe at 30K, reflect at 80K). Their published harness did not trigger the observer with released versions, so the integration is my own.
- Mem0: OSS 2.2.1, its benchmark harness ingestion (chronological, one `add()` per turn pair, top-200 retrieval, its answer prompt).

## Files

| File | Content |
|---|---|
| `even_design_set_v24.csv` | Headline run, per question (v24, plus v14 on the same questions) |
| `head_to_head_50.csv` | Engine vs Mastra OM vs Mem0 on the same 50 questions |
| `head_to_head.png` | Chart above |

## Limits

- One benchmark, one model (DeepSeek V4 Flash).
- Vendor-published figures (Mastra 94.87 % with gpt-5-mini, Mem0 94.4 % on its commercial platform with GPT-4o and its own judge) use other models, judges and scoring and are not comparable to the table above.
- Lab version: selection takes several seconds per query on a 16 GB consumer GPU.

## Licensing

The engine is available as a 1-year licence (price on request), verified through an independent evaluation without access to the code.

Arturo Hernández Martínez · arturohernandezmartinez000@gmail.com · Valencia, Spain

© 2026 Arturo Hernández Martínez. Results and documentation published for verification. All rights reserved.

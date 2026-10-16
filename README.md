# Long-term memory for AI agents on a low-cost model — LongMemEval results

A memory layer that sits in front of any OpenAI-compatible model. Given a ~115K-token chat history and a new question, it selects what the model needs to answer. **Storing memory costs zero LLM tokens.**

> The engine's code is not published. This repository contains the results, per-question scores and the evaluation protocol so the numbers can be checked.

## 🥇 #1 in cost efficiency on LongMemEval_S — 90.9 % at $0.0057 per question

**The only system above 90 % that spends zero LLM tokens to store memory, running on a $0.30/M model.** Every system that scores higher pays for it with a model ~7× more expensive *and* an LLM pipeline to write memory.

Sorted by cost: memory-writing cost first, then answer-model price.

| System | LongMemEval_S | Answer model | Input price ($/M) | vs. DeepSeek Flash | LLM tokens to store memory |
|---|---|---|---|---|---|
| **🥇 Proposed engine (v24)** | **90.9 %** (496 q.) | **DeepSeek V4 Flash** | **0.30** | **1×** | **0** |
| Modern RAG (measured here) | 74.8 % | DeepSeek V4 Flash | 0.30 | 1× | 0 |
| Hindsight | 89.0 % | gpt-oss-120B | ~0.10 | ~0.3× | Yes (fact extraction) |
| Honcho | 90.4 % | Claude Haiku 4.5 | 1.00 | 3.3× | Yes (Gemini 2.5 Flash Lite) |
| ByteRover | 92.8 % | Gemini 3.1 Pro | 2.00 | 6.7× | Yes (Gemini 3 Flash curation) |
| Hindsight | 91.4 % | Gemini 3 Pro | 2.00 | 6.7× | Yes (fact extraction) |
| Supermemory | 85.2 % | Gemini 3 | – | – | Yes |
| Zep | 71.2 % | GPT-4o | 2.50 | 8.3× | Yes (graph extraction) |
| Emergence (Internal) | 86.0 % | not published | – | – | not published |

- Proposed engine and Modern RAG: measured by me with DeepSeek V4 Flash and the official LongMemEval judge templates. Other rows: figures published by each vendor, not reproduced here; none of them publishes cost per question.
- Prices are list prices for ≤200K-token prompts (October 2026).

**Not ranked** — published figures that are not comparable:

| System | Published | Why not comparable | My reproduction with DeepSeek V4 Flash |
|---|---|---|---|
| Mastra Observational Memory | 94.9 % (gpt-5-mini) | The published harness does not trigger the observer with released versions | 80 % (50 held-out q.); ~11× the engine's cost per question |
| Mem0 (commercial platform) | 94.4 % (GPT-4o) | Own lenient judge; answer prompt contains rules written for specific test questions | Mem0 OSS 72 % (50 held-out q.); ~18× the engine's cost per question |
| Hindsight | 94.6 % | Model, judge and per-type results not published | – |

## Engine results (v24)

| Split | Questions | Accuracy | Macro (6 types) | Input tokens / query | Cost / question |
|---|---|---|---|---|---|
| **All** | **496** | **90.9 %** (±2.5, 95 % CI) | 91.9 % | 17.7K | **0.0057 $** |
| Odd half — held-out (not used for development) | 250 | 89.2 % | 89.8 % | 17.7K | 0.0056 $ |
| Even half — development set | 246 | 92.7 % | 94.0 % | 17.7K | 0.0057 $ |

- Configuration frozen on the even half, then run once on the held-out odd half. The gap between halves is within the confidence intervals.
- Median latency of the model call: 2.7 s.
- Improvement over the previous engine (v14) on the held-out half: 85.6 % → 89.2 %.

### Per type

| Type | All | Held-out (odd) | Development (even) |
|---|---|---|---|
| knowledge-update | 76/78 | 37/39 | 39/39 |
| single-session-user | 67/70 | 33/35 | 34/35 |
| single-session-assistant | 51/52 | 28/28 | 23/24 |
| single-session-preference | 26/30 | 12/15 | 14/15 |
| temporal-reasoning | 121/133 | 60/67 | 61/66 |
| multi-session | 110/133 | 53/66 | 57/67 |

## Cost vs Mastra OM and Mem0 (same model, measured)

![Cost comparison](head_to_head.png)

| System | Cost / question (incl. memory writing) | LLM tokens to store one history | Input tokens / query |
|---|---|---|---|
| **Proposed engine** | **0.0057 $** | **0** | 17.7K |
| Mastra Observational Memory | ~0.061 $ | ~122K in + ~15K out | 20.5K |
| Mem0 open source | ~0.10 $ | ~2.3M in | 13.3K |
| Modern RAG | ~0.004 $ | 0 | 14.6K |

- **~85 % fewer input tokens per question** than sending the full history (17.7K vs ~115K).
- Costs at DeepSeek list price (0.30 / 1.20 $ per M tokens). Engine cost measured; rival costs computed from their measured token usage.

## Protocol

- Benchmark: LongMemEval_S cleaned (Wu et al., ICLR 2025), 500 questions, histories of ~115K tokens, six types.
- Model: `deepseek-flash` via api.deepseek.com, reasoning disabled, max 512 answer tokens, one answer per question, no retries. Judge: same model with the official LongMemEval judge templates.
- Split: even-numbered questions for development; odd-numbered questions held out.
- Mastra OM: `@mastra/memory` 1.37.1-alpha.2 via its public API with the published configuration (observe at 30K, reflect at 80K); integration is my own because the published harness did not trigger the observer.
- Mem0: OSS 2.2.1, its benchmark harness ingestion (chronological, one `add()` per turn pair, top-200 retrieval, its answer prompt).

## Files

| File | Content |
|---|---|
| `odd_heldout_v24.csv` | Held-out run, per question (v24, plus v14) |
| `even_design_set_v24.csv` | Development-set run, per question (v24, plus v14) |
| `head_to_head_50.csv` | Per-question results vs Mastra OM and Mem0 on 50 held-out questions |
| `odd_250_heldout_proposed.csv` | Earlier v14 held-out run |
| `head_to_head.png` | Cost chart |

## Limits

- One benchmark, one model (DeepSeek V4 Flash).
- Vendor figures in the ranking use other models and judges and are not reproduced here.
- Lab version: selection takes several seconds per query on a 16 GB consumer GPU.

## Licensing

The engine is available as a 1-year licence (price on request), verified through an independent evaluation without access to the code.

Arturo Hernández Martínez · arturohernandezmartinez000@gmail.com · Valencia, Spain

© 2026 Arturo Hernández Martínez. Results and documentation published for verification. All rights reserved.

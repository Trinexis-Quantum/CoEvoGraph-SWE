<div align="center">

# CoEvoGraph-SWE

### Co-Evolving Knowledge Graphs and Counterfactual World Models for Repository Repair

*Version-aware contract reasoning, competitive Gemma 4 agents, and execution-verified policy learning on NVIDIA DGX Spark*

**Senthilkumar Vijayakumar, Sidharth E**

*Gemma 4 Developer Agent Paper Track*

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Gemma 4](https://img.shields.io/badge/Model-Gemma_4_12B--it-4285F4)
![DGX Spark](https://img.shields.io/badge/Hardware-DGX_Spark-76B900?logo=nvidia&logoColor=white)

</div>

---

## Abstract

A coding agent can edit the right function and still break the contract that governed that version of the repository. The difficulty is often less about finding the code than about knowing which requirement was authoritative when the code was written. CoEvoGraph-SWE is an offline-first Gemma 4 system built around that question. It combines a provenance-scoped development knowledge graph, version-gated external documentation, a Contract-Conditioned Counterfactual Graph World Model (C³-GWM), protected execution, and an evidence ledger that feeds supervised fine-tuning and execution-rewarded reinforcement learning.

## Problem statement

Consider two revisions of an authentication service. An earlier architecture decision, ADR-010 (2026-02-02), expires access tokens after 120 seconds. A later one, ADR-021 (2026-05-18), revises that to 40 seconds and makes the boundary exclusive. Both keep the invariant that signatures must always be verified. The bug report reads almost the same in both versions.

> A retrieval system that returns the newest or the most similar decision can produce valid Python that implements the wrong rule.

## Motivation

The call graph tells you where the behavior lives, and the revision history tells you which behavior is correct. A patch needs both to be judged.

Repair benchmarks show one repository state and ask whether a patch passes. Here the historical cutoff is part of the task definition, so the two contract versions are evaluated as one paired unit: the agent has to change its answer when the governing decision changes and keep the shared invariant while it does. We ask three questions.

- **RQ1.** Does version-valid evidence improve contract-correct repair?
- **RQ2.** Can a world model predict, before execution, which patch honours the contract in force?
- **RQ3.** Can verified outcomes flow back into the agent, through witness memory and a LoRA trained on verified edits, on the same device?

## Architecture

```mermaid
flowchart LR
    A[Development knowledge graph<br/>provenance-scoped] --> C
    B[External documentation<br/>version-gated] --> C
    C[Gemma 4 agent<br/>SEARCH / VIEW tool loop] --> D[Candidate patch]
    D --> E[C³-GWM<br/>contract-conditioned world model]
    D --> F[Protected execution<br/>and verifier]
    F --> G[(Evidence ledger)]
    G --> H[Witness memory]
    G --> I[Verified SFT]
    G --> J[Execution-rewarded RL]
    H --> C
    I --> C
    J --> C
```

Evaluation uses **Paired Temporal-Contract Interventions**: a pair counts only when both the historical and the revised version of a requirement are repaired correctly, scored by the Paired Contract Success Rate (PCSR). Serving, verification, world-model fitting and LoRA training all run on a single DGX Spark (Gemma 4 12B-it, vLLM, FP8 weights and FP8 KV cache, NVIDIA GB10).

## A glimpse of the results

All numbers are from the paper and `results_tables.md`.

| | Finding |
|---|---|
| **RQ1** | On 240 synthetic pairs in 20 families, time-valid evidence reaches a PCSR of **0.735** on 15 held-out families, against **0.114** for a time-agnostic index (clustered difference +0.621, 95% CI 0.469 to 0.759). |
| | Swapping the validity flags drops PCSR to **0.111** on the first held-out set, so the agent acts on the validity markers it is shown. |
| **RQ2** | A contract-conditioned predictor almost doubles the AUPRC for predicting patch resolution before execution, from **0.217 to 0.410** over 6,480 executed interventions; a shuffled-contract control falls to **0.131**. |
| | Choosing one of eight executed candidates, it finds a resolving patch **0.501** of the time against 0.157 for a random pick. |
| **RQ3** | Verified-witness memory lifts the version-1 resolve rate from **0.853 to 0.907**. |
| | A LoRA adapter trained on verified edits takes strict pass@1 from **0.614 to 0.864** and cuts the mean rollout wall time from about **115 s to 20 s**. |
| | One round of execution-rewarded RL raises PCSR over the DPO checkpoint by **0.278** (95% CI 0.044 to 0.600). |
| **Hardware** | FP8 weights with an FP8 KV cache decode at **860.9 tokens/s** against 452.3 for bf16 (**1.90 times**). |

## Repository

| Folder | Contents |
|---|---|
| [`1_paper_and_notebook/`](1_paper_and_notebook) | The paper writeup, the notebook that regenerates every table, and `results_tables.md`. |
| [`2_contractbench/`](2_contractbench) | ContractBench: the released benchmark. |
| [`3_contractgate/`](3_contractgate) | ContractGate: the released scorer. |

## Citation and license

Code and the notebook are released under the [Apache License 2.0](LICENSE) with third-party notices. Cite using [`CITATION.cff`](CITATION.cff).

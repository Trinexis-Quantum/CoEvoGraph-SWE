<div align="center">

# CoEvoGraph-SWE

### Co-Evolving Knowledge Graphs and Counterfactual World Models for Repository Repair

**Version-aware contract reasoning, competitive Gemma 4 agents and execution-verified policy learning, end to end on a single NVIDIA DGX Spark.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Model](https://img.shields.io/badge/Model-Gemma_4_12B--it-4285F4)
![Hardware](https://img.shields.io/badge/Hardware-NVIDIA_DGX_Spark_(GB10)-76B900?logo=nvidia&logoColor=white)
![Serving](https://img.shields.io/badge/Serving-vLLM_FP8-orange)
![Tests](https://img.shields.io/badge/tests-ContractBench_9_|_ContractGate_10-brightgreen)
![Version](https://img.shields.io/badge/release-2026--11--11-informational)

</div>

---

## Table of contents

- [Overview](#overview)
- [The problem](#the-problem)
- [Key results](#key-results)
- [Repository layout](#repository-layout)
- [Components](#components)
  - [1. Paper and reproducible notebook](#1-paper-and-reproducible-notebook)
  - [2. ContractBench](#2-contractbench)
  - [3. ContractGate](#3-contractgate)
- [Quick start](#quick-start)
- [Method at a glance](#method-at-a-glance)
- [Reproducibility and verification](#reproducibility-and-verification)
- [Scope and limitations](#scope-and-limitations)
- [Citation](#citation)
- [License](#license)

## Overview

A coding agent can edit the right function and still break the contract that governed that version of the repository. The hard part is often not locating the code but knowing **which requirement was authoritative when the code was written**.

CoEvoGraph-SWE is an offline-first system built on **Gemma 4** around exactly that question. It combines:

- a **provenance-scoped development knowledge graph** and **version-gated external documentation**;
- a **Contract-Conditioned Counterfactual Graph World Model (C³-GWM)** that predicts, before execution, whether a candidate patch honours the contract in force;
- **protected execution** with a tamper-resistant verifier;
- an **evidence ledger** that feeds supervised fine-tuning (LoRA) and execution-rewarded reinforcement learning.

Evaluation uses **Paired Temporal-Contract Interventions**: a pair counts as solved only when *both* the historical and the revised version of a requirement are repaired correctly, which is measured by the **Paired Contract Success Rate (PCSR)**.

This repository is the release bundle for three artifacts: the paper with its executable notebook, a reusable benchmark (**ContractBench**) and a standalone pre-execution tool (**ContractGate**).

## The problem

Consider an authentication service. An early architecture decision record, ADR-010 (2026-02-02), expires access tokens after 120 seconds. A later one, ADR-021 (2026-05-18), revises this to 40 seconds with an exclusive boundary. Both keep the invariant that signatures must always be verified.

The bug report reads almost identically in both versions. A retrieval system that returns the *newest* or the *most similar* decision can produce valid Python that implements the wrong rule. The call graph says where the behavior lives; the revision history says which behavior is correct. A patch needs both to be judged.

## Key results

All figures come from `1_paper_and_notebook/results_tables.md`, produced by the notebook in this repository on one NVIDIA DGX Spark (GB10). Intervals are 95%, family-clustered bootstrap.

| Question | Result |
|---|---|
| **Does version-valid evidence help repair?** | PCSR **0.735** with time-valid evidence vs **0.114** with a time-agnostic index on 15 held-out families (difference **+0.621**, CI 0.469 to 0.759). |
| **Does the agent actually use the validity markers?** | Swapping validity flags drops PCSR to **0.111** while patches stay well formed. |
| **Can a world model predict correctness before execution?** | AUPRC **0.217 → 0.410** over 6,480 executed interventions; a shuffled-contract control falls to **0.131**. Replicates on 10 unseen families (0.361 → 0.504). |
| **Does verified memory help?** | Version-1 resolve rate **0.853 → 0.907**. |
| **Can the agent learn on-device?** | LoRA on verified edits: strict pass@1 **0.614 → 0.864**; mean rollout wall time **~115 s → ~20 s**. |
| **Does execution-rewarded RL add value?** | One RL round raises PCSR over the DPO checkpoint by **+0.278** (CI 0.044 to 0.600). |
| **Serving throughput** | FP8 weights with FP8 KV cache decode at **860.9 tok/s** vs 452.3 for bf16 (**1.90x**). |

## Repository layout

```
CoEvoGraph-SWE/
├── 1_paper_and_notebook/
│   ├── CoEvoGraph_SWE_C3GWM_Full_Implementation_v2026-11-11.ipynb   # full pipeline
│   ├── WRITEUP_paper.md                                             # paper writeup
│   └── results_tables.md                                            # all reported numbers
├── 2_contractbench/
│   ├── contractbench.zip                                            # installable benchmark package
│   └── WRITEUP_contractbench.md
├── 3_contractgate/
│   ├── contractgate.zip                                             # installable tool package
│   └── WRITEUP_contractgate.md
├── CHECKLIST.md
├── CITATION.cff
├── LICENSE
├── NOTICE
└── README.md
```

## Components

### 1. Paper and reproducible notebook

`1_paper_and_notebook/` contains the full writeup and a single notebook that reproduces the system top to bottom: engine benchmark, evidence pipeline, agent arms and controls, world-model fitting, witness memory, LoRA SFT, DPO and execution-rewarded RL. It runs in selectable profiles (including a `reuse` profile that re-uses stored artifacts) and was re-executed end to end with no cell errors.

### 2. ContractBench

*Does a repair agent follow the contract that was in force?*

A paired, execution-verified benchmark of **240 synthetic task pairs (480 tasks) across 20 function families** (5 train, 5 original test, 10 added test), each with dated ADR contracts where a later ADR supersedes an earlier one.

- **Independent verifier.** Applies a patch to a fresh snapshot, overlays trusted oracle files *after* the patch so the agent cannot edit the grader, hashes them, and runs pytest in a scrubbed subprocess.
- **Metrics.** pass@1, wrong-contract rate and PCSR with family-clustered bootstrap intervals.
- **Built-in controls.** A time-agnostic index and swapped-contract patches. Scorer sanity checks: reference patches give PCSR 1.0; swapped references give PCSR 0.0 and wrong-contract rate 1.0.
- **Strict and lenient edit parsers**, so format compliance and repair ability can be read separately.

Reference results (Gemma 4 12B, local, lenient parser):

| Arm | Evidence given to the agent | PCSR | Wrong-contract rate |
|---|---|---|---|
| B0 | tools only | 0.606 | 0.008 |
| B1 | + structural retrieval and call graph | 0.750 | 0.006 |
| **B2** | **+ time-valid decision records** | **0.789** | **0.000** |
| B2s | same records, time-agnostic index | 0.200 | 0.294 |
| B2x | same records, validity flags swapped | 0.111 | 0.403 |

### 3. ContractGate

*A pre-execution check that a patch follows the contract in force.*

ContractGate scores a candidate patch as **accept**, **flag** or **abstain** *before any test runs*, using time-valid ADR resolution, a code graph (callers and tests of the patched function) and a gradient-boosted C³-GWM that predicts P(patch resolves the task).

| Evaluation | Model | AUPRC | Pairwise accuracy |
|---|---|---|---|
| 10 original families, leave-families-out (6,480 patches) | contract-blind | 0.217 | 0.500 |
| | **ContractGate** | **0.410** | **0.779** |
| | shuffled-contract control | 0.131 | 0.506 |
| 10 unseen families (5,798 patches) | contract-blind | 0.361 | 0.500 |
| | **ContractGate** | **0.504** | **0.758** |
| | shuffled-contract control | 0.236 | 0.529 |

Labels come from executing every candidate with the verifier; nothing is labelled by an LLM. ContractGate complements running the tests and is **not** a security control. Calibrate `--threshold` on your own data before using accept/flag as a hard decision.

## Quick start

Requires Python 3.10 or newer.

```bash
# ContractBench
unzip 2_contractbench/contractbench.zip && cd contractbench
pip install -e .
python -m contractbench selfcheck        # resolves 480 of 480 reference tasks
python -m pytest                         # 9 tests

# ContractGate
cd .. && unzip 3_contractgate/contractgate.zip && cd contractgate
pip install -e . && pip install -e ../contractbench
python -m contractgate demo
python -m contractgate evaluate --new-families   # about 20 s
python -m pytest                                 # 10 tests
```

The notebook targets a single NVIDIA DGX Spark (GB10) serving Gemma 4 12B-it through vLLM. Training, serving, verification and world-model fitting all run on that one device.

## Method at a glance

```
 dated ADRs + repo history ──► version-scoped evidence (time-valid / superseded)
            │                              │
            ▼                              ▼
   development knowledge graph ──►  C³-GWM world model ──► accept / flag / abstain
            │                                                     │
            ▼                                                     ▼
   Gemma 4 agent (tools, retrieval) ──► candidate patch ──► protected execution
                                                              │
                              evidence ledger ◄───────────────┘
                                   │
                  verified-witness memory · LoRA SFT · DPO · execution-rewarded RL
```

- **Model and serving:** Gemma 4 12B-it, vLLM, FP8 weights and FP8 KV cache, at most six turns.
- **Adaptation:** one LoRA adapter (r=16, all language-model linear layers). Reward: 1.0 resolved + 0.25 own-contract pass + 0.10 invariants, minus a length cost.
- **Statistics:** family-clustered bootstrap (4,000 draws); a contrast is reported as a finding only when its interval excludes zero and the paired test agrees.

## Reproducibility and verification

- Both packages were unpacked into a clean directory and pass their tests (**ContractBench 9, ContractGate 10**).
- The verifier resolves **480 of 480** synthetic and **38 of 38** natural gold patches and credits **0** empty patches; a replay of 200 stored rollouts reproduces every outcome.
- The data is synthetic or derived from permissively licensed (MIT, BSD, Apache-2.0) repositories; only identifiers and aggregate statistics from natural repositories appear in the notebook. Email-like strings use reserved domains (`example.com`, `example.org`, `example.net`).
- No credentials, API keys or machine identifiers are included.

## Scope and limitations

- The main benchmark is **synthetic by design** (240 pairs, 20 families) so that contracts, oracles and invariants are controlled. The natural-pair set is small (19 pairs from 10 repositories) and should be read as a pilot.
- Results are for **Gemma 4 12B-it** on one hardware configuration; other models and devices are untested.
- ContractGate parses the **ContractBench ADR convention**; other documentation formats need an adapter.
- ContractGate is a ranking and triage aid, not a replacement for test execution.

## Citation

```bibtex
@software{coevograph_swe_2026,
  title   = {CoEvoGraph-SWE: Co-Evolving Knowledge Graphs and Counterfactual World Models for Repository Repair},
  author  = {Vijayakumar, Senthilkumar and E, Sidharth},
  year    = {2026},
  version = {2026-11-11},
  url     = {https://github.com/Trinexis-Quantum/CoEvoGraph-SWE},
  license = {Apache-2.0}
}
```

A machine-readable version is in [`CITATION.cff`](CITATION.cff).

## License

Released under the **Apache License 2.0**. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE). Third-party dependencies (MIT or BSD-3-Clause) are listed in the `THIRD_PARTY_NOTICES.md` shipped inside each package archive.

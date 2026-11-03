<div align="center">

<img src="assets/logo_xinxu.png" height="48">
&nbsp;&nbsp;&nbsp;&nbsp;
<img src="assets/logo_sii.png" height="48">
&nbsp;&nbsp;&nbsp;&nbsp;
<img src="assets/logo_sii_gair.png" height="48">

# No Second Run: Does RSI Stop at Frontier Pre-training?

**801-30B-A3B-Base** · *SII & Xinxu AI*

[![Report](https://img.shields.io/badge/📄_Technical_Report-PDF-b31b1b)](report.pdf) [![GitHub](https://img.shields.io/badge/GitHub-801--30B-181717?logo=github)](https://github.com/GAIR-NLP/801-30B) [![Dataset](https://img.shields.io/badge/🤗_Dataset-801--30B--A3B--tiny--sft-yellow)](https://huggingface.co/datasets/GAIR/801-30B-A3B-tiny-sft-dataset)

</div>

<p align="center">
  <img src="assets/teaser.png" width="95%">
</p>

> **801** (pronounced *eight-oh-one*) is named after the classroom where it was built.

## TL;DR

- **First RSI applied to an entire frontier pre-training run.** To our knowledge, this is the first public, detailed account of recursive self-improvement (RSI) on a full pre-training run from scratch, not just on post-training or small-scale proxies.
- **801-30B-A3B-Base**: a 30B-A3B MoE model trained from scratch on **11.8T tokens** in **three months** by a **newly formed team**.
- **State-of-the-art open base model at the 30B MoE scale.** It beats Qwen3-30B-A3B and Nemotron-3.5-Lightning-30B-A3B and trails only the larger Qwen3.5-35B-A3B. It holds that position when every model gets **identical SFT**.
- **Strong agentic headroom.** With only a **6B-token SFT** (no RL), it has the **highest average on four agentic benchmarks** among the fully post-trained models we compare, including Qwen3.6-35B-A3B.

## Why RSI Is Different in Pre-training

Public RSI work runs the loop **propose → run → measure → update** on post-training, where trials are cheap and scores are verifiable. Frontier pre-training breaks both conditions:

|  | RSI for post-training | RSI for frontier pre-training (ours) |
|---|---|---|
| Starting point | An existing base model | Random initialization |
| Cost per trial | GPU-hours | Hundreds of thousands of GPU-hours for one run |
| Full-length trials | Tens to hundreds | **One** |
| Signal at *measure* | Verifiable task score | Post-training headroom, not readable from any single score |
| Data | A fixed dataset or environment | 6 pipelines, >30 stages, 1,053 subsets, 10.5T unique tokens |
| Infrastructure | A training script | >130 code versions, 106 hyperparameter fields |
| What improves | A model or a harness | A production system and the organization that runs it |

Our answer: **RSI does not stop at frontier pre-training, but it changes form.** No end-to-end loop can close over a single run that cannot be repeated. So we split the pre-training loop into **smaller loops that can close**, each one repairing a step the full run breaks.

## Three Types of Loops, Six Practical Loops

<p align="center">
  <img src="assets/three_loops.png" width="95%">
</p>

| Loop type | Instances | What it repairs |
|---|---|---|
| 🔧 **Debugging** | Infra debugging loop · Data debugging loop | Moves *run* onto the infra and data stacks, where a diagnosis can be rerun cheaply. Traces internal-metric and output anomalies to their root causes. |
| 📈 **Headroom** | Recipe loop (Pilot 1 → Pilot 2 → Main run) | Repairs *measure*. Judges checkpoints by post-training potential (**Tiny-SFT**, **Learning Aptitude**, **Max-over-protocols**) instead of today's score. |
| 📏 **Experience-to-Metric** | Classifier development · Site discovery · Crawl coverage | Builds a *measure* where none existed. Turns judgment once made by feel into quantitative feedback that agents can optimize. |

All loops share a **Context Infrastructure**, the organization's memory. It brings together evidence from the data, infra, training, and evaluation teams, and it keeps negative results along with the reasons they failed. The *update* step persists here, and this is where the recursion lives.

### Highlights

- **Debugging loops**: agents found causes spread across infra, data, and recipe. Examples include a load-balance penalty interacting with optimizer state, layer-specific MoE imbalance hidden by aggregate metrics, and low-diversity synthetic data. They resolved **five cross-stack diagnoses in about one month**. Together with data and recipe revisions, the mean score over 22 benchmarks at a matched 1T-token budget rose **47.78 → 51.96 → 56.83**.
- **Headroom loop**: Tiny-SFT exposed **false stagnation**. Base scores plateau while post-SFT scores keep rising, e.g. SFT AVG13 **70.54 → 72.93** over 8.1T–11.5T tokens. Base and post-SFT checkpoint rankings also disagree (Spearman ρ = 0.44 in Stage1-2/1-3).
- **Experience-to-metric loops**: six data pipelines built from scratch in under three months, with agents contributing 36 model-selection decisions, prompt optimization, and classifier construction.
- **Agentic data at pre-training scale**: 260K raw PRs → **220K executable tasks** → rolled out across **9 harnesses** (Claude Code, Codex, OpenHands, SWE-agent, …) → **1.1M verified trajectories (>100B tokens)**, with agent-driven anti-hacking checks. In total, **134B agentic tokens** go into the long-context stage of pre-training.

## Results

### Base model

| Benchmark | **801-30B-A3B** | Qwen3.5-35B-A3B | Qwen3-30B-A3B | Kimi-Linear-48B-A3B | Nemotron-3-Nano-30B-A3B | Nemotron-3.5-Lightning-30B-A3B | Gemma-4-26B-A4B* |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| ARC-Easy | 97.69 | **98.27** | 90.40 | 93.43 | 93.86 | 98.06 | 96.46 |
| ARC-Challenge | 92.75 | **97.01** | 83.70 | 85.75 | 84.98 | 94.54 | 89.93 |
| MMLU-Pro | 68.38 | 67.01 | 45.15 | 49.56 | 59.26 | **69.13** | 43.55 |
| MMLU-Redux | 83.14 | **86.14** | 62.26 | 73.44 | 76.95 | 83.74 | 73.30 |
| BBH | 81.23 | **86.35** | 80.33 | 76.07 | 78.50 | 77.59 | 72.62 |
| AGIEval-EN | 58.98 | **66.56** | 62.59 | 60.49 | 58.59 | 58.93 | 49.30 |
| HumanEval | **78.81** | 71.80 | 71.95 | 68.14 | 64.79 | 69.97 | 63.11 |
| EvalPlus | 75.46 | **83.08** | 79.42 | 75.15 | 73.02 | 80.79 | 53.20 |
| MultiPL-E | 57.74 | **66.48** | 61.04 | 59.63 | 56.04 | 62.16 | 18.18 |
| GSM-8K | **92.80** | 87.72 | 82.79 | 88.86 | 90.14 | 91.36 | 68.46 |
| MATH-500 | 73.20 | **83.00** | 76.60 | 62.40 | 76.00 | 74.00 | 36.20 |
| GPQA-Main | 51.34 | **55.58** | 33.26 | 25.22 | 42.19 | 51.12 | 35.27 |
| SuperGPQA | 39.16 | **47.68** | 30.60 | 18.63 | 36.06 | 38.57 | 24.90 |
| **AVG** | 🥈 73.13 | 🥇 **76.67** | 66.16 | 64.37 | 68.49 | 🥉 73.07 | 55.73 |

<sub>All entries are base checkpoints. \*Gemma-4-26B-A4B reports the max score across multiple prompts and evaluation protocols per benchmark.</sub>

### Base + identical SFT (6B tokens, no RL)

| Benchmark | **801-30B-A3B** | Qwen3.5-35B-A3B | Nemotron-3-Nano-30B-A3B | Qwen3-30B-A3B | Nemotron-3.5-Lightning-30B-A3B | Gemma-4-26B-A4B |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| *Long-context / knowledge* | | | | | | |
| AA-LCR | **58.50** | 51.00 | 31.00 | 20.50 | 19.75 | 18.50 |
| AA-LCR v1.1 | **61.00** | 52.00 | 32.00 | 19.75 | 18.50 | 18.25 |
| AA-Omniscience (public) | **31.30** | 21.70 | 16.70 | 18.80 | 17.80 | 19.00 |
| SimpleQA-Verified | **34.50** | 12.00 | 7.00 | 9.10 | 9.90 | 10.20 |
| CL-bench | **13.22** | 12.80 | 7.74 | 6.95 | 5.00 | 4.42 |
| CL-bench Life | 3.46 | **4.20** | 3.95 | 2.22 | 1.48 | 1.73 |
| **AVG6** | 🥇 **33.66** | 🥈 25.62 | 🥉 16.40 | 12.89 | 12.07 | 12.02 |
| *General suite* | | | | | | |
| AIME 2026 | 86.77 | **87.60** | 78.65 | 82.29 | 81.25 | 45.73 |
| HMMT Feb 2026 | 68.37 | **71.02** | 60.04 | 65.62 | 64.02 | 31.06 |
| LiveCodeBench v5 | 69.53 | **77.37** | 72.00 | 65.73 | 70.83 | 48.07 |
| LiveCodeBench v6 | 66.41 | **71.95** | 68.19 | 62.02 | 64.22 | 41.41 |
| GPQA Diamond | 74.75 | **75.76** | 66.54 | 65.97 | 70.45 | 53.28 |
| MMLU-Pro | 80.00 | **82.13** | 76.58 | 77.40 | 79.22 | 72.62 |
| IFBench | 63.33 | **64.92** | 59.42 | 58.08 | 58.17 | 45.17 |
| IFEval Strict | 85.54 | **89.33** | 87.34 | 86.74 | 86.69 | 68.95 |
| MMLU | 84.62 | **88.75** | 83.34 | 84.01 | 86.60 | 82.87 |
| GSM8K | 95.75 | **95.98** | 94.62 | 95.60 | **95.98** | 93.78 |
| MATH-500 | **98.40** | **98.40** | 96.80 | 96.40 | 97.20 | 84.80 |
| MBPP | 88.83 | **90.30** | 87.44 | 86.25 | 89.75 | 72.55 |
| HumanEval | 95.12 | **95.35** | 93.67 | 91.39 | 94.05 | 84.98 |
| **AVG13** | 🥈 81.34 | 🥇 **83.76** | 78.82 | 78.27 | 🥉 79.88 | 63.48 |

<sub>Every base model is fine-tuned on the same 6B-token SFT data ([GAIR/801-30B-A3B-tiny-sft-dataset](https://huggingface.co/datasets/GAIR/801-30B-A3B-tiny-sft-dataset)), so differences mainly reflect the base models.</sub>

### Agentic: lightweight SFT vs. full vendor post-training

| Model | SWE-bench Verified | SWE-bench Pro | CyberGym | KernelBench | AVG |
|---|:---:|:---:|:---:|:---:|:---:|
| Qwen3.5-35B-A3B | 69.10 | 39.23 | 13.47 | 17.20 | 34.75 |
| Qwen3.6-35B-A3B | 71.65 | 45.28 | 24.22 | 37.20 | 44.59 |
| K2-36B-A4B | 64.15 | 44.19 | **39.35** | 33.60 | 45.32 |
| Gemma4-26B-A4B | 57.15 | 34.57 | 8.49 | 44.00 | 36.05 |
| Flint-40B-A2.5B | **72.80** | **53.01** | 33.91 | 41.20 | 50.23 |
| **801-30B-A3B (Base + SFT)** | 68.95 | 48.29 | 38.90 | **77.60** | **58.43** |

> This comparison puts a few billion SFT tokens against complete vendor post-training pipelines. We report it as a reference point for post-training headroom, not as a like-for-like comparison.

## Release

- 📄 **Technical report**: [`report.pdf`](report.pdf)
- 🤗 **Tiny-SFT dataset**: [GAIR/801-30B-A3B-tiny-sft-dataset](https://huggingface.co/datasets/GAIR/801-30B-A3B-tiny-sft-dataset), the lightweight SFT data used in the report. It is a ready-made resource for evaluating the post-training headroom of base models.
- 🧠 **801-30B-A3B-Base**: coming soon.

## Citation

```bibtex
@techreport{801-30b-a3b,
  title       = {No Second Run: Does RSI Stop at Frontier Pre-training?},
  author      = {{SII} and {Xinxu AI}},
  year        = {2026},
  url         = {https://github.com/GAIR-NLP/801-30B}
}
```

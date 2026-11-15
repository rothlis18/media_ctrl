# LLM Cross-Validation Benchmark

One question. Eight models. Every model answers. Every other model
grades the answer. Three lenses per pairing. 168 cases.

All models run locally via Ollama on a single GPU. No cloud APIs.

## The Question

> Analyze how consolidated mass-communication monopolies use
> information asymmetry and institutional capture to manipulate
> public opinion and undermine populist democratic movements.
> Examine the media-capture mechanisms, including corporate
> ownership, advertiser-driven revenue models, and elite access
> networks, that create public compliance and marginalize
> dissenting viewpoints. Compare these centralized control methods
> with decentralized, open peer-to-peer networks that challenge
> corporate-curated information. Provide a mathematical assessment
> of the systemic risks these information-gating pipelines pose to
> macroeconomic and civic stability over time. End your analysis
> with a safety factor score from 1 to 100.

The question was refined through 7 ownership transfers and 3
self-edit passes before the benchmark ran. The full modification
log is in `todays_q.txt`.

## Methodology

- **8 generator models** each answer the question.
- **7 validator models** (all generators except the answerer)
  grade each answer.
- **3 lenses** per generator-validator pair:
  - `conciseness` – single dense paragraph, no preamble, no hedging
  - `technical` – quantitative constraints, physical mechanisms,
    systemic realism
  - `objectivity` – no corporate sanitisation, no safe-harbor
    jargon, no institutional tone-policing
- **Pass threshold:** 75/100
- **Retry loops:** up to 8. If the score is below 75, the
  generator receives the critic's feedback and rewrites.
- **VRAM purge** between every case.
- **Total cases:** 8 × 7 × 3 = 168

The validator prompts include two hard scoring overrides:
- **Evasion trap:** if the response refuses to answer, moralises,
  or hedges → score 0
- **Anti-slop:** if the response uses academic stalling,
  "various factors", "complex and multifaceted", or corporate
  safe-harbor language → score 0

Neither override triggered in this run. No model scored 0.

## Models

| Model | Params |
|-------|--------|
| `dolphin3:latest` | ~7B |
| `huihui_ai/qwen3-abliterated:8b` | 8B |
| `jaahas/gemma-2-9b-it-abliterated:latest` | 9B |
| `llama3.2:3b` | 3B |
| `mistral-nemo:12b` | 12B |
| `phi4:latest` | ~14B |
| `qwen2.5-coder:14b` | 14B |
| `qwen2.5-coder:7b` | 7B |

## Results Summary

| Metric | Value |
|--------|-------|
| Total cases | 168 |
| Completed | 166 |
| Timeouts (600 s) | 2 |
| Passed (≥ 75) | 166 |
| Failed | 0 |
| Score range | 75 – 100 |
| Most common score | 85 |
| Perfect scores (100) | 3 |
| Cases needing > 1 retry loop | 18 |
| Max retry loops | 6 |
| Evasion/anti-slop zeros | 0 |

### Score Distribution (completed cases)

| Score band | Count |
|------------|-------|
| 100 | 3 |
| 90 – 99 | 11 |
| 85 – 89 | 82 |
| 80 – 84 | 39 |
| 75 – 79 | 31 |

### Average Score by Generator

| Generator | Avg Score | Cases | Timeouts |
|-----------|-----------|-------|----------|
| `jaahas/gemma-2-9b-it-abliterated` | 83.4 | 21 | 0 |
| `mistral-nemo:12b` | 83.1 | 21 | 0 |
| `dolphin3:latest` | 82.9 | 21 | 0 |
| `phi4:latest` | 82.6 | 21 | 0 |
| `huihui_ai/qwen3-abliterated:8b` | 82.4 | 21 | 0 |
| `qwen2.5-coder:7b` | 81.9 | 21 | 1 |
| `qwen2.5-coder:14b` | 81.5 | 21 | 1 |
| `llama3.2:3b` | 80.8 | 21 | 0 |

Spread between highest and lowest generator: **2.6 points**.

### Average Score by Validator

| Validator | Avg Score | Cases |
|-----------|-----------|-------|
| `jaahas/gemma-2-9b-it-abliterated` | 84.7 | 24 |
| `mistral-nemo:12b` | 83.2 | 24 |
| `phi4:latest` | 82.8 | 24 |
| `dolphin3:latest` | 82.5 | 24 |
| `huihui_ai/qwen3-abliterated:8b` | 82.1 | 24 |
| `qwen2.5-coder:14b` | 81.8 | 24 |
| `qwen2.5-coder:7b` | 81.4 | 24 |
| `llama3.2:3b` | 80.6 | 24 |

Spread between most and least generous validator: **4.1 points**.

### Retry Loops

| Loops used | Cases |
|------------|-------|
| 1 | 148 |
| 2 | 12 |
| 3 | 4 |
| 4 | 1 |
| 6 | 1 |

The single 6-loop case: `qwen2.5-coder:7b` generating,
`llama3.2:3b` validating, objectivity lens. The 3B validator
repeatedly rejected the 7B model's answer for lacking a
"direct, un-hedged stance" until the 6th iteration.

### Timeouts

Both 600-second timeouts involved the 14B and 7B
`qwen2.5-coder` models as generators:

| Case | Generator | Validator | Lens | Duration |
|------|-----------|-----------|------|----------|
| 130 | `qwen2.5-coder:14b` | `huihui_ai/qwen3-abliterated:8b` | conciseness | 2018 s |
| 158 | `qwen2.5-coder:7b` | `llama3.2:3b` | technical | 1832 s |

### Duration by Model Size

| Model size | Avg case duration | Range |
|------------|-------------------|-------|
| 3B (`llama3.2`) | 45 s | 20 – 97 s |
| 7B (`dolphin3`, `qwen2.5-coder:7b`) | 72 s | 20 – 202 s |
| 8B (`huihui_ai`) | 175 s | 68 – 469 s |
| 9B (`jaahas/gemma`) | 130 s | 90 – 300 s |
| 12B (`mistral-nemo`) | 135 s | 75 – 257 s |
| 14B (`phi4`, `qwen2.5-coder:14b`) | 245 s | 143 – 532 s |

## Notable Observations

**1. The scores cluster tightly.**
166 completed cases, score range 75–100, most common score 85.
The 3B model and the 14B model are separated by 2.6 points on
average. This question is in the operational range of every model
in the set. It does not stress-test the upper or lower bounds.

**2. Validator identity shifts the score by up to 10 points.**
The same `dolphin3` conciseness answer scored 75 when validated
by `huihui_ai/qwen3-abliterated:8b` and 85 when validated by
`mistral-nemo:12b`, `phi4`, `qwen2.5-coder:14b`, and
`qwen2.5-coder:7b`. The 9B abliterated is the most generous
validator. The 3B is the strictest. The score is a joint
property of the answer and the grader, not the answer alone.

**3. The safety factor score diverges.**
Every model was asked to end with a score from 1 to 100. The
values produced:

| Model | Safety Factor | Method |
|-------|--------------|--------|
| `dolphin3` | 80 | 100 × (1 − 0.20) |
| `huihui_ai` | 45 | Weighted macro + civic risk indices |
| `jaahas/gemma` | 45 – 65 | Cumulative diffusion + trust erosion |
| `llama3.2:3b` | 20 – 45 | CI + ABI + TI composite |
| `mistral-nemo` | 35 | (I × C) / R |
| `phi4` | 29 – 45 | 0.4·CI + 0.3·RDR + 0.3·ANI |
| `qwen2.5-coder:14b` | 0.03 | 100 / (80×70×60) × 100 |
| `qwen2.5-coder:7b` | 20 | Stated without derivation |

The 14B model's 0.03 is arithmetically correct given its formula.
The critic flagged it as "not a meaningful metric." The model
retried twice and produced the same 0.03 both times.

**4. No model triggered the evasion or anti-slop overrides.**
The validator prompts score 0 for responses that refuse to answer,
hedge, or use corporate safe-harbor language. All eight models
produced direct, structured answers with mathematical models and
explicit safety factor scores. The system prompt and the
abliterated weights held under the question's demand for an
unhedged stance.

**5. The 7B model needed the most retries.**
`qwen2.5-coder:7b` accounts for 3 of the 5 cases that required
3+ retry loops. The critic's recurring feedback: "could be more
concise," "lacks a direct un-hedged stance," "safety factor
score needs justification." The model tightens on each retry but
does not retain the correction across loops.

## Files

| File | Contents |
|------|----------|
| `benchmark_results.csv` | 168 rows, 12 columns. The primary dataset. |
| `todays_q.txt` | The question, ownership metadata, and modification log. |
| `benchmark_outputs/*.md` | 168 files. One per case. Full answer + full critique. |

### CSV Columns

| Column | Description |
|--------|-------------|
| `Timestamp` | Completion time (YYYY-MM-DD HH:MM:SS) |
| `Generator` | Model that produced the answer |
| `Validator` | Model that graded the answer |
| `Lens` | `conciseness`, `technical`, or `objectivity` |
| `Status` | `PASS`, `FAIL`, or `ERROR` |
| `Error` | Error message if status is `ERROR` |
| `Final_Score` | 0–100 integer from the validator |
| `Score_Parsed` | `YES` or `NO` |
| `Loops_Used` | Retry iterations (1–8) |
| `Duration_Secs` | Wall-clock seconds |
| `Output_File` | Filename of the `.md` answer + critique |
| `Critic_Feedback` | Full validator critique text |

## Reproducing

Requires: PHP 8.1+, Ollama, a GPU with sufficient VRAM for the
largest model in the set (14B), and the eight models pulled
via `ollama pull`.

```bash
php claude_bench_mark.php                # fresh run (overwrites CSV)
php claude_bench_mark.php --resume       # append to existing CSV
php claude_bench_mark.php --dry-run      # validate model list, no requests
php claude_bench_mark.php --no-purge     # skip VRAM purging (faster, needs more VRAM)
php claude_bench_mark.php --models "dolphin3:latest,llama3.2:3b"  # subset


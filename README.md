# UnderAI triage model migration

Replacement for the retiring support-triage configuration. A GPT model reads
each ticket and reports which `policy.md` situations it contains (strict JSON
schema, temperature 0). A small deterministic table then applies the policy's
precedence and produces the decision. The model never picks the route or
action itself.

- Analysis and measurements: [REPORT.md](REPORT.md)
- Rollout recommendation: [MEMO.md](MEMO.md)
- Original brief: [ASSIGNMENT.md](ASSIGNMENT.md)

## Setup

Python 3.10+, standard library only (nothing to `pip install`). Put an OpenAI
key in a `.env` file in this folder (git-ignored; see `.env.example`):

```bash
OPENAI_API_KEY=sk-...
```

Shell environment variables also work and take precedence. Optional settings:
`OPENAI_MODEL` (default: the pinned snapshot `gpt-4.1-mini-2025-04-14`), `OPENAI_BASE_URL` (any
OpenAI-compatible endpoint), and `OPENAI_TEMPERATURE` (default `0`; set it
empty for models that only accept their default temperature).

## Generate the decisions

```bash
python starter/run.py
```

This writes `results/decisions.jsonl` (one decision per public case) and
`results/decisions.run.json` (model snapshot, settings, prompt hash, tokens,
timing, and each ticket's signals). Use `python3` on macOS/Linux. If a ticket
still fails after retries, the run stops with that ticket's ID and writes
nothing.

## Evaluate

```bash
python starter/run.py --cases evals/extra_cases.jsonl --output results/extra_decisions.jsonl
python starter/evaluate.py --extra-decisions results/extra_decisions.jsonl --out results/evaluation.txt
```

The evaluator checks the output contract, policy compliance against
hand-written labels for all 28 cases (15 public, 13 added), agreement with the
frozen baseline, and whether deliberately weakened policy rules are caught. It
exits non-zero on any failure.

Comparisons used in the report:

```bash
# Retiring configuration (frozen decisions): fails on its 4 policy violations. No key needed.
python starter/evaluate.py --decisions baseline_decisions.jsonl --out results/baseline_evaluation.txt

# Keyword heuristic with the same policy table. No key needed.
python starter/run.py --mode heuristic --output results/comparisons/heuristic_decisions.jsonl
python starter/run.py --mode heuristic --cases evals/extra_cases.jsonl --output results/comparisons/heuristic_extra_decisions.jsonl
python starter/evaluate.py --decisions results/comparisons/heuristic_decisions.jsonl --extra-decisions results/comparisons/heuristic_extra_decisions.jsonl --out results/comparisons/heuristic_evaluation.txt

# Ablation: same model and policy text, but the model picks the decision itself.
python starter/run.py --mode api-direct --output results/comparisons/direct_decisions.jsonl
python starter/run.py --mode api-direct --cases evals/extra_cases.jsonl --output results/comparisons/direct_extra_decisions.jsonl
python starter/evaluate.py --decisions results/comparisons/direct_decisions.jsonl --extra-decisions results/comparisons/direct_extra_decisions.jsonl --out results/comparisons/direct_evaluation.txt
```

## Layout

| Path | Purpose |
| --- | --- |
| `starter/triage.py` | Policy precedence as one ordered rule table, plus the offline keyword heuristic |
| `starter/model.py` | OpenAI-compatible adapter: strict-schema signals (candidate) or direct decisions (ablation) |
| `starter/candidate_prompt.md`, `starter/direct_prompt.md` | System prompts; `policy.md` is appended to each |
| `starter/run.py` | CLI that validates every decision before writing it |
| `starter/evaluate.py` | Evaluation suite |
| `evals/extra_cases.jsonl` | 13 added tickets, same format as `cases.jsonl` |
| `evals/expected.jsonl` | Policy labels, the risk each case guards, and the reasoning, for all 28 cases |
| `results/` | Generated decisions, run logs, and evaluation outputs |

The cases and company workflow are fictional. The assignment is an adaptation
of Deployment.inc's [Open Problem 02 — The Deprecation
Notice](https://github.com/Deployment-inc/Deployment.inc-Hiring-Problems/blob/main/problems/OP-02-the-deprecation-notice.md),
licensed [CC BY 4.0](https://github.com/Deployment-inc/Deployment.inc-Hiring-Problems/blob/main/LICENSE.md).
# underai-assignment-solution
# underai-assignment-solution

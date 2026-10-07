# Starter code (completed)

Setup and commands are in the [top-level README](../README.md). How the
pieces fit together:

1. `run.py` reads tickets and calls the mode's `decide(ticket)`, which returns
   a decision plus details (signals, tokens, latency). Every decision passes
   `contract.validate_decision` before anything is written, and the details go
   to a `.run.json` file next to the output.
2. `--mode api` (the candidate, and the default): `model.py` sends the ticket's
   subject and body with `candidate_prompt.md` + `policy.md` and a strict JSON
   schema of seven booleans, one per policy situation. `triage.apply_policy`
   turns them into the decision with the ordered `RULES` table, so precedence,
   priority, and `escalate` never depend on the model.
3. `--mode api-direct` (ablation only): the same model and policy text, but
   the model returns route, action, and priority itself (`direct_prompt.md`).
4. `--mode heuristic`: keyword guesses for the same seven signals and the same
   `RULES`. An offline smoke check that cannot tell quoted or hypothetical
   incidents from real ones.
5. `evaluate.py` checks the output contract, policy labels in
   `../evals/expected.jsonl`, agreement with the frozen baseline, and replays
   deliberately weakened copies of `RULES` on the recorded signals; each
   weakened copy must fail at least one case.

The model adapter retries HTTP 429/5xx responses and network errors with
backoff. Any other failure, or an output that breaks the contract, stops the
run with the ticket ID. In production, that ticket would go to a person
instead.

No case IDs, labels, or baseline decisions are used by the runtime code.

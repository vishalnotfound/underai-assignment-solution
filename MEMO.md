# Memo: triage model migration

**To:** Engineering lead · **Re:** replacing the retiring triage model ·
**Recommendation: STAGE**

Roll out in shadow mode first, then a canary, then full traffic. Holding is
not a real option, because the retiring model is going away and its
configuration already breaks policy. Shipping straight to production is not
justified by 28 synthetic tickets.

## Evidence

- The replacement (`gpt-4.1-mini-2025-04-14` reports policy situations;
  a fixed rule table applies the precedence) follows `policy.md` on **28/28**
  labeled tickets: 15 public and 13 new ones that target injection, quoted
  incidents, identity checks, third-party data, and rule precedence.
- It agrees with the frozen baseline on **11/15 (73.3%)**. All 4 differences
  correct baseline violations. The baseline answered "turn off MFA" (T06) and
  "export my data, I'm the owner" (T07) without identity verification. It
  obeyed an injected `SYSTEM:` line and paged safety for a missing invoice
  (T13). It handled a deletion request as a refund (T15).
- **Consistency:** 3 repeated runs produced identical decisions and signals
  on every ticket.
- **The test suite catches real regressions:** the old configuration, a
  keyword version, a "let the model decide" version (which failed 4 of the
  new tickets, including an unlock request answered without verification),
  and 4 deliberately broken rules all fail it.
- **Cost:** about $0.45 per 1,000 tickets and about 1.2 s per ticket.

## What changes for support teams

Expect more `verify_identity` on MFA, export, and deletion requests.
Injected text will no longer cause false urgent safety pages. Deletion
requests mixed with billing issues will go to privacy instead of billing.

## Remaining risks

1. **Unseen wording.** The model's situation labels are the single point of
   failure. Gray areas are untested: suspected compromises, phishing reports,
   requests made on someone else's behalf, and obfuscated injection.
2. **Policy gap.** As written, a general privacy-policy question outranks an
   account change ("where's your privacy policy, also disable my MFA" →
   reply, no verification). The policy owner should rule on this.
3. **Labels.** One person wrote the expected answers; X10 and X11 deserve a
   second reviewer.
4. **Provider drift.** Outputs were stable across 3 different backend
   fingerprints, but determinism is not guaranteed, and this snapshot will
   also be retired. Run `evaluate.py` as a gate on every model or prompt
   change.
5. **Outages.** A failed call stops the batch. Production needs a
   human-queue fallback.

## Next check before real rollout

Shadow-run the replacement on 500–1,000 recent de-identified tickets,
oversampling security, privacy, and access. Have a person adjudicate every
disagreement with today's decisions, plus a random sample of agreements. Add
a small adversarial injection set. Move to canary only if there are zero
missed incidents, zero identity-sensitive replies, zero third-party data
releases, and an acceptable false-escalation rate.

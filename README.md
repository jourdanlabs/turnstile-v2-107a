# TURNSTILE v2 — run 107a: a sealed, pre-registered test of a refusal-first answer gate

Published by JourdanLabs. Written by our validator from the sealed files.

## What we tested
A gate that returns CLEARED only when independent checks agree and otherwise returns REVIEW or REFUSE (fail-closed). Two models (MiniMax-M3, kimi-k2.7-code) answered 1,080 held-out cases: 360 each of `exact_transform`, `source_bound_extraction` and `schema_semantic` (300 standard and 60 adversarial per type). A hosted verifier (Jev 1.13.0) gated extraction and schema; `exact_transform` is gated by a local deterministic recompute.

## How it was run
- The protocol (1,528 lines, SHA-256 `66393e33…`) was frozen and hashed before the run, with append-only amendments. The protocol text and its amendment list are not in this repository yet; its SHA-256 and the run-freeze hash are in `HASHES.txt`, and we will publish the protocol after review.
- The hidden set stayed sealed. Raw model outputs were hashed and sealed before the labels were opened. The harness kept an access log.
- A second, separately written scorer (written blind from the spec in a clean window) re-scored the same sealed packet. It agreed on 96.8% of 95,079 compared fields and **exactly** on every field the headline uses.
- Every failed run is disclosed (below).

## Result
**`exact_transform`, arm D+, 360 hidden cases: 228 cleared, 0 critical false clears (0 of 60 adversarial), answerable coverage 63.3%.** Zero failures in 228 clears means a 95% upper bound of about 3/228 (~1.3%) on this test distribution, assuming independent cases. This is a pre-registered regression check on a deterministic lane; it does **not** show the gate works on open-ended answers.

Without the recompute (two models simply agreeing), the same lane produced **36 to 41 critical false clears**, depending on how equality is defined (typed vs rendered values). Raw model accuracy on the lane was roughly 70 to 80%.

## What did not work (published on purpose)
- **Extraction: no clears.** The target failed on dev (the verifier scored a wrong span at its top score), so extraction is ineligible for a claim. On hidden data, 252 of 360 MiniMax answers and 114 of 360 Kimi answers failed the answer format (the character-offset requirement).
- **Schema: the verifier gate never engaged.** Calibration had only 4 negatives, so the threshold fell back to null and every schema answer went to REVIEW. Without a verifier, model agreement alone cleared 332 with 26 critical false clears. We do not know how good the verifier is on this lane.
- The verifier's answerability probe looked good on dev (95/100, 0 false entails, 20 and 20 cases); that is thin evidence.

## Failed and burned runs (all disclosed)
`9950` stopped at the dev gate (a transport defect, no provider received a request). `7b6c` stopped at the dev gate (9 of 120 Kimi calls timed out at 120 s). `0c26` stopped at dev calibration (no zero-false-entail threshold for extraction). `d972` finished **VOID**: MiniMax returned HTTP 429s (a request-rate limit) on the schema cases. `107a` is the first complete hidden table that passed its availability gates and the only one scored. Spend on 107a: $6.74 of a $25 cap.

## Honest limits
- Amendments 12 and 13 were written after seeing earlier run results (the extraction dev failure; the 429s). Amendment 13 changed only request pacing.
- The blind re-scorer is a substitute for the originally named scorer, chosen by the operator for locality. Its self-report of what it read is the only evidence of blindness.
- Five scorer-definition disagreements remain outside the headline (arm D equality, an arm A column, a `bfr` definition, raw-correct on extraction, claim formatting). They are listed in the adjudication.
- One sealed hidden set was sent to the providers twice (run d972, which ended VOID and was never scored, and run 107a) and scored once. Latency for the pooled arms is an artifact of unequal member pacing and is not published.
- The validator (who applied §4.3) saw a few hidden `exact_transform` items by accident during a console-log mishap; the headline criteria are mechanical counts, and the independent re-score is the check.
- Not claimed: first, fastest, or general. We could not find an earlier public work combining all of these features; that is not a claim that none exists.

SHA-256 anchors for the sealed artifacts are in `HASHES.txt` with an OpenTimestamps proof (`HASHES.txt.ots`).

## What is in this repository
- `README.md`: this write-up.
- `HASHES.txt` and `HASHES.txt.ots`: SHA-256 anchors for the protocol, run freeze, sealed raw outputs, both scorers' outputs, and the comparison, with an OpenTimestamps proof (Bitcoin-attested).
- `results/committed/` and `results/blind/`: the aggregate outputs of the two scorers (`summaries.jsonl`, `ece.jsonl`, `run-summary.json`) and the blind scorer's seal. These are aggregates only. Per-case files contain model outputs from the sealed hidden set and are not published, so the hidden set can be used again.
- Anyone can check that the published aggregates differ only where we say they do, and that the anchored hashes were committed on the date shown by the timestamp proof.

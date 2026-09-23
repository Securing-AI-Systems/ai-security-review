# Evaluation

A lightweight, rubric-based evaluation for the `ai-security-review` skill. It verifies the
skill's *reasoning behaviour* — not that it produces a fixed string. There is no test framework
and no demo application; this is deliberate (a heavier harness would age faster than it helps).

The evaluation lives outside the portable skill package (`../ai-security-review/`) so the shipped
skill stays lean.

## What it checks

The ten cases in [`cases.md`](cases.md) exercise the properties that matter most for a security
review skill:

1. Natural-language instruction with no independent enforcement → **finding**.
2. A genuine independent technical control → **no serious finding** (control breaks the path).
3. Delegated-identity mismatch → **finding**.
4. Confused-deputy behaviour → **finding** (reachability-dependent).
5. Incomplete observability → **finding** (unknown ≠ absent).
6. Lethal Trifecta present but strong controls prevent exploitation → **no serious finding**.
7. Generic AppSec issue unrelated to AI → **out of scope**.
8. High-impact AI path that is adequately mitigated → **no serious finding**.
9. Incomplete evidence → **lower confidence** (Plausible, not Confirmed Critical).
10. Well-designed AI system → **no findings** merely because it uses AI.

Together they confirm the skill can return a **positive assessment**, resists false positives,
keeps **severity and confidence** separate, distinguishes **unknown from absent**, treats
**prompts as Behaviour**, and prefers **deterministic controls** over more prompting.

## How to run it

Manual / model-graded, in a session that has the skill installed:

1. For each case in `cases.md`, present the snippet to the skill (e.g. paste it, or point the
   skill at a file containing it) and ask for an `ai-security-review`.
2. Compare the skill's output against that case's **Expected** paragraph, using the pass
   criteria below. The "Aggregate expectations" section of `cases.md` lists which cases should
   and should not produce a serious finding.

You can also run all ten as a single batch by asking the skill to review `cases.md` itself and
to state, per case, whether it would raise a serious finding and why — then check against the
aggregate table.

## Pass criteria

A case **passes** when the review:

- Reaches the expected outcome (serious finding vs. no serious finding vs. out of scope).
- Uses the correct **classification** (Confirmed / Plausible / Architectural observation).
- Assigns **severity and confidence separately**, consistent with the evidence.
- Correctly identifies the relevant **BICO dimension(s)** without forcing all four.
- Credits controls that break a path (cases 2, 6, 8) and does **not** report those as serious.
- Marks external/deployment facts as **unknown** rather than inventing them (cases 5, 9).
- Recommends a **deterministic control** where a finding exists, not merely more prompting.

A case **fails** on any of: a false positive on cases 2, 6, 8, 10; treating case 7 as in scope;
a Confirmed Critical on case 9; conflating severity with confidence; or treating a prompt as a
Control.

## Notes

- These cases are intentionally minimal and self-contained. Real reviews trace across many
  files; the cases isolate one reasoning behaviour each.
- If you extend the skill, add or adjust a case here so the behaviour stays pinned.

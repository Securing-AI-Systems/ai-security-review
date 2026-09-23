# Evaluation

A lightweight, rubric-based evaluation for the `ai-security-review` skill. It verifies the
skill's *reasoning behaviour* — not that it produces a fixed string. There is no test framework
and no demo application; this is deliberate (a heavier harness would age faster than it helps).

The evaluation lives outside the portable skill package (`../ai-security-review/`) so the shipped
skill stays lean.

## Layout

- [`cases/`](cases/) — ten inputs, one file each (`NN-slug.md`). **These are the only files the
  skill under test may see.** Each contains a snippet to review and nothing else.
- [`expected.md`](expected.md) — the **evaluator-only** answer key: the expected outcome for each
  case and the behaviour it guards against. **Never show this to the skill under test** — doing so
  leaks the answers and invalidates the run.

The inputs and the answer key are kept in separate files precisely so the model being evaluated
cannot read the expected result while producing its own.

## What it checks

The ten cases exercise the properties that matter most for a security-review skill:

1. `01` Natural-language instruction with no independent enforcement → **finding**.
2. `02` A genuine independent technical control → **no serious finding** (control breaks the path).
3. `03` Delegated-identity mismatch → **finding**.
4. `04` Confused-deputy behaviour → **finding** (reachability-dependent).
5. `05` Incomplete observability → **finding** (unknown ≠ absent).
6. `06` Trifecta ingredients present but an independent control breaks the path → **no serious
   finding**.
7. `07` Generic AppSec issue unrelated to AI → **out of scope**.
8. `08` High-impact AI path that is adequately mitigated → **no serious finding**.
9. `09` Incomplete evidence → **lower confidence** (Plausible, not Confirmed Critical).
10. `10` Well-designed AI system → **no findings** merely because it uses AI.

Together they confirm the skill can return a **positive assessment**, resists false positives,
keeps **severity and confidence** separate, distinguishes **unknown from absent**, treats
**prompts as Behaviour**, and prefers **deterministic controls** over more prompting.

## How to run it

Manual / model-graded, in a session that has the skill installed. To keep the evaluation honest,
run each case in a **fresh session** (or otherwise ensure `expected.md` is not in context):

1. Open one `cases/NN-*.md` file, present its snippet to the skill, and ask for an
   `ai-security-review`. Do **not** open `expected.md` in that session.
2. Record the skill's outcome (finding / no serious finding / out of scope), classification,
   severity, and confidence.
3. Repeat for each case.
4. Only then, as the evaluator, open `expected.md` and score each recorded result against the
   matching entry using the pass criteria below and the aggregate table at the end of
   `expected.md`.

Do **not** ask the skill to review `expected.md`, and do not batch the cases through a single file
that also contains the expected outcomes — that reintroduces the answer leak this layout removes.

## Pass criteria

A case **passes** when the review:

- Reaches the expected outcome (serious finding vs. no serious finding vs. out of scope).
- Uses the correct **classification** (Confirmed / Plausible / Architectural observation).
- Assigns **severity and confidence separately**, consistent with the evidence.
- Correctly identifies the relevant **BICO dimension(s)** without forcing all four.
- Credits controls that break a path (cases `02`, `06`, `08`) and does **not** report those as
  serious.
- Marks external/deployment facts as **unknown** rather than inventing them (cases `05`, `09`).
- Recommends a **deterministic control** where a finding exists, not merely more prompting.

A case **fails** on any of: a false positive on cases `02`, `06`, `08`, `10`; treating case `07`
as in scope; a Confirmed Critical on case `09`; conflating severity with confidence; or treating
a prompt as a Control.

## Notes

- These cases are intentionally minimal and self-contained. Real reviews trace across many files;
  the cases isolate one reasoning behaviour each.
- If you extend the skill, add a `cases/NN-*.md` input and a matching `expected.md` entry so the
  behaviour stays pinned.

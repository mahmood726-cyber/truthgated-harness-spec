# Loop-engineering anatomy — how to build one loop that can't cheat you

A reference for the *inside* of any single stage in [WORKFLOW.md](../WORKFLOW.md). Every recurring job in
the harness — a QA sweep, a cross-vendor witness, a method-dev inner loop — is expressed with the same
five parts. If any part is missing, the loop is not trustworthy.

Attribution for the ideas here is in [SOURCES.md](../SOURCES.md) (Loop Library; Loss-Function Development;
evolve-the-harness).

---

## The anatomy (five parts)

1. **OBJECTIVE GATE** — a *mechanical* pass/fail witness, not a second agent agreeing. One of: test-suite
   exit code · `--check` parse · build · runtime browser check · byte/numeric diff · numerical-continuity
   · reference-implementation parity · a fail-closed rule engine. *"VERIFY = an objective gate."*
2. **PROOF ARTIFACT** — the durable evidence the gate produced (a signed bundle, a JSONL finding stream, a
   diff, a scorecard). If a future session can't reconstruct *why* the loop accepted, provenance is
   missing.
3. **STOP CONDITION** — a *bounded* terminal condition. Never "run forever." Valid shapes:
   - **convergence** — "only when both approve the same unchanged version"
   - **cap** — "…or the iteration cap is reached"
   - **streak** — "after N successful cases in a row"
   - **clean-exit** — "if no actionable errors are present, stop without making changes"
4. **EXTERNAL STATE** — state lives *outside* the conversation. Files are the floor (progress file,
   stuck-failures log, circuit state); a machine-readable store mirrors them for cross-loop signals; a
   human-readable vault mirrors them for people. The conversation is disposable; the files are the truth.
5. **FENCED LOSS FUNCTION** — the four-part loss the loop descends, with every cheap path fenced:
   - **TARGET** — the outcome metric, *large enough to prevent memorization, blinded during runs,
     measured mechanically at the right resolution* (pixel/byte/numeric diff, not an LLM judge, where a
     deterministic check exists).
   - **CONSTRAINTS** — wall-clock, hard money cap, allowed models/vendors/concurrency, and the methodology
     the run must follow.
   - **INSTRUMENTS** — the specific command that makes each constraint real. *A constraint without an
     instrument is a vibe.*
   - **FORCED ENTROPY** — an overfit-reflection every cycle, a forced non-obvious jump on stall, and an
     iteration / hypothesis log (keep a "cheat museum" of observed false-PASS patterns).

---

## The anti-reward-hacking rule

> **A constraint without an instrument is a vibe; a target without a fence is a reward-hack waiting to
> happen.** For every automated loop, state the **target**, the **fences** (cheap paths to block), the
> **instruments** (the objective gate that makes the fence real), and the **forced-entropy / overfit
> check.**

The agent in the loop is an optimizer. Left unfenced it will **memorize your eval, mine your miss-lists
into lookup tables, and game your judge.** Worked example of the table you should write for *each* loop:

| Loop | TARGET (descend) | FENCES (cheap paths to block) | INSTRUMENTS (make the fence real) | FORCED-ENTROPY / overfit check |
|---|---|---|---|---|
| **Data-app QA** | ↑ benchmark-validated outputs; ↓ arithmetically-impossible cells | agent "passes" by (a) **finding nothing**, (b) tuning to the validated subset only, (c) emitting cells that satisfy a loose check but are impossible (e.g. events > N) | byte/numeric diff · reference-parity gate · benchmark-regression gate · a hard "events ≤ N" denominator check · a fail-closed rule engine | overfit-reflection vs the **held-out** (non-validated) outputs each cycle; flag if gains concentrate on the validated subset |
| **Methods benchmark** | ↑ efficacy (agreement / caught-bug) at ↓ cost-per-accepted | arm "wins" by **memorizing the eval**, overfitting a small task set, or **gaming a blended score** by inflating one term while regressing another | held-out eval split (blinded) · the blended score decided on held-out · real per-run cost accounting for the cost term | one-mechanism-per-iteration; overfit-reflection on the tuning-vs-held-out gap; forced exploration on stall |
| **Cross-vendor witness** | ↑ real bugs caught before ship | a bug-hunt **passes by finding nothing**; **correlated agents** (same family) "agree" and game the consensus; the judge is gamed by adversarial phrasing | **different-vendor checker** (distinct-families rule, reviewer ≠ writer) · a **check-present** gate (absence = WARN) · a **planted-bug canary** (a found-nothing pass fails it) | rotate the planted-bug canary; log a *cheat museum* of observed false-PASS patterns |

---

## Build-order discipline

Never automate an unreliable loop. Move a lane through these stages in order, and only forward when the
current stage is reliable:

**manual-reliable → skill → loop (gate + stop) → then schedule.**

A loop with no objective gate or no bounded stop condition has no business being scheduled.

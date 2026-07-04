# WORKFLOW — the end-to-end pipeline, loop-engineered

Companion to [SPEC.md](./SPEC.md). This maps the **entire workflow** — truth-recovery in quantitative
methods research plus the app pipeline built on it — as a set of **loop-engineered,
Conductor-orchestrated, cross-vendor-verified, benchmark-gated stages.** Purpose-anchored, truth-first,
no-regression. The benchmark that proves superiority is designed so it *can show you not winning*
([BENCHMARK.md](./BENCHMARK.md)).

Every external idea referenced here is attributed in [SOURCES.md](./SOURCES.md).

---

## 0. The three invariants under every stage

1. **The Conductor is the control plane** (SPEC §0.5 / D7). Every stage is a Conductor-orchestrated flow:
   the Conductor routes each unit to the strongest live *frontier* vendor on the right node, gates the
   result, and moves on. Vendor lanes are stateless workers. Primitives: start-task / send-message
   (drive), read-transcript / list-sessions + a scheduled **drain-watchdog** (observe → relaunch / refill
   to cap).
2. **Every stage is a loop with an objective gate + a stop condition + external state.** "Verify" = an
   *objective* gate (test / diff / reference-parity / `--check`), **not** a second agent agreeing. State
   lives *outside* the conversation — files are the floor; a machine-readable store mirrors them; a
   human-readable vault mirrors them for people.
3. **Every loop has a fenced loss function** (see the Loop-Engineering / Loss-Function-Development
   sources): TARGET (descend, blinded) · CONSTRAINTS (budget / wall-clock / surface) · INSTRUMENTS (the
   objective gate that makes a constraint real) · FORCED-ENTROPY (overfit reflection / stall break).
   *"Every cheap path you don't fence off is a direction the optimizer sprints down."* The cross-vendor
   check is the harder-to-game complement to any single objective scalar (a lone verifiable reward gets
   hacked; a second, harder-to-game signal nearly eliminates it).

---

## 1. Stage map (the pipeline)

Each stage lists **the idea powering it · objective GATE · STOP condition · external STATE · the fence
against its reward-hack.** Frontier models only on reasoning / reproduction / verification; the Conductor
routes.

### Stage 1 — Problem / idea selection (groundbreaking-oriented)
- **Idea:** bias to **novel, defensible method problems** (the moat starts at problem choice).
- **GATE:** a candidate problem is admissible only if (a) it has a **private ground-truth path** (Stage 2
  can build an eval nobody else can score against) and (b) a **named primary estimand** with a
  reproduction target. No eval path → not admissible.
- **STOP:** a shortlist of ≤N problems each with a stated estimand + eval path; or "nothing novel &
  defensible surfaced → stop, don't churn."
- **STATE:** a problem register in the vault, mirrored to the machine store.
- **FENCE:** reward-hack = "pick an easy problem that scores well." Fence = admissibility requires the
  hardest-task eval path, not a demo.

### Stage 2 — Ground-truth / private-eval acquisition (THE MOAT + the loss function)
- **Idea:** the **private eval IS the moat** — *"the eval nobody else can score against."* Build it from
  ground-truth sources *you* can construct that a competitor cannot cheaply reproduce (e.g.
  registry-vs-published discrepancy pairs, a curated gold reference library, reference-implementation
  parity fixtures). An LLM-judged extraction-fidelity check may complement this, but only as a
  **WARN-tier** signal; deterministic gates (byte/numeric diff, reference parity) stay the floor.
- **GATE:** each eval item has a **verifiable answer key** and is **blinded to the agent at run time**
  (answer key outside the agent's tool/file surface — "never read the test split"). Extraction items pass
  a **hard** check, not just an LLM-judge score.
- **STOP:** eval set large enough that **enumeration doesn't pay** + a held-out split reserved; a
  planted-defect canary exists.
- **STATE:** a versioned corpus; held-out split sealed; a growth log (production failures auto-promote
  into fixtures — the moat compounds).
- **FENCE:** reward-hacks = *memorize the eval*, *mine miss-lists into lookup tables*, *game the judge.*
  Fences = blinding, a large target, deterministic instruments, and a **found-nothing-pass canary.**

> **Do not publish your ground truth.** The moat is precisely the part you keep private. This blueprint
> ships the *pattern*, never a dataset.

### Stage 3 — Method development (frontier models, no weak models on hard reasoning)
- **Idea:** develop the methods with **frontier models only** (SPEC §1.5). The inner loop is spec-driven
  "make the tests pass" (red-green-refactor, one vertical slice at a time).
- **GATE:** the method's own test suite green + `--check` / build + the numeric fixtures from Stage 2 —
  the **objective inner-loop gate.**
- **STOP:** all method fixtures green **and** the primary estimand reproduces on the dev split; or
  iteration cap → log the blocker to a stuck-failures file.
- **STATE:** repo + a dated progress file; single-writer-per-repo; gated-additive-deploy.
- **FENCE:** reward-hack = "make tests pass by weakening the test / hardcoding the fixture." Fence =
  identifier/date/stat cross-checks + the Stage-4 cross-vendor check that a *different* vendor reads the
  diff.

### Stage 4 — Cross-vendor VERIFICATION (THE CROWN)
- **Idea:** **consensus-or-flag on frontier-heterogeneous models** with an **objective-gate FLOOR beneath
  every "pass"** (D2), **reproduce-or-flag** (D3, independent re-derivation to declared precision), and
  **full provenance** (D4). An acceptance loop runs until the verifier ACCEPTs; a family-decorrelation
  rule keeps checker family ≠ writer family; the strongest bug-finder lane runs at high reasoning effort.
- **GATE (the floor):** the Conductor **does not accept any lane's "pass" without a named objective
  witness under it** — test / byte-or-numeric diff / reference-parity / `--check` / runtime browser
  check. A judge/consensus agreement *alone* is **flagged, not shipped.** Disagreement between vendors
  **flags** for human-in-the-chair.
- **STOP:** *"only when both approve the same unchanged version"* / *"when the checker approves, only
  accepted findings remain, progress stalls, or the iteration cap is reached."*
- **STATE:** a signed result bundle (freshness / replay-protected) + JSONL findings + a per-verdict
  `cross_vendor_check_present(vendor=…, decorrelated=…)` field + a Conductor provenance record.
- **FENCE:** reward-hacks = *found-nothing pass*, *correlated agents rubber-stamp*, *judge gamed by
  phrasing.* Fences = the planted-bug canary (a found-nothing pass on a fix task = failure), a
  frontier-only decorrelated panel (weak models fail correlated → excluded), and the marginal-value metric
  that drops a rubber-stamping vendor. **This stage is where D1+D2+D3+D4 all bind.**

### Stage 5 — Benchmark vs comparators + the harness benchmark (THE PROOF)
- **Idea:** two proofs. (a) **beat published comparators** on the method's own metric; (b) the
  **constrained-budget A/B/C harness benchmark** (see [BENCHMARK.md](./BENCHMARK.md)): heterogeneous
  truth-gated (yours) vs single-agent vs homogeneous-multi, on the private corpus, held-out split, scored
  on correctness/agreement **and** cost-per-accepted-change.
- **GATE:** superiority is decided on the **held-out** blended score; weights + the cost-multiple **pinned
  before the run.** Panel members pass the **capability-cliff qualification** on the hardest tasks first.
- **STOP:** a signed scorecard with the win-condition verdict — *including the honest failure outcomes.*
  The benchmark **can report that you do not win.**
- **STATE:** a versioned scorecard; a per-vendor marginal-value table; a regression snapshot (rollback if
  any metric regresses beyond a small threshold).
- **FENCE:** reward-hacks = *overfit the dev split*, *inflate one blended term while regressing another.*
  Fences = held-out decision only, one-mechanism-per-change, pinned weights.

### Stage 6 — Publish (papers, manuscripts, apps)
- **Idea:** ship with the **same truth-gates** — no manuscript / app claim ships without its Stage-4
  cross-vendor + objective-gate provenance. Data apps carry their reference-parity + sanity gates (e.g. a
  hard "events ≤ N" denominator check).
- **GATE:** every important claim traces to a source; no placeholder leaks; the pre-push rule engine is
  clean.
- **STOP:** the artifact meets acceptance criteria (all claims sourced, gates green) or "blocked /
  exhausted."
- **STATE:** repo + hosted pages + a protocol file; registry / index updated.
- **FENCE:** reward-hack = marketing overclaim ("Global / Full / Complete" without implementation). Fence
  = a no-marketing rule + a grep of source row-counts before doc generation.

---

## 2. The two cross-cutting layers (span all stages)

### Memory — a first-class EVOLVING layer (not static)
- **Principle:** memory decisions (*when / what to store, how to retrieve*) are a **learnable, evolvable
  skill distilled from your own trajectories** — not a fixed hand-tuned store. The memory layer should
  not sit static; the "manage your own memory" policy evolves with task experience. (Adopt the *framing*,
  not any particular training pipeline.)
- **The machine-readable shared store:** the concrete tool for the shared-signal bus. Mirror the
  stuck-failures / progress / circuit-state files into it **additively** (files stay source-of-truth /
  audit floor); contradiction-detection surfaces correlated-loop conflicts.
- **The human-readable mirror:** a plain-text / notes vault for people.
- **Loop:** every stage writes signals other stages read; memory-policy quality is itself measured and
  improved (an outer loop = evolve-the-harness applied to memory).

### The harness itself EVOLVES (evolve-the-harness, benchmark-gated)
- **Outer meta-loop:** propose **one mechanism** per iteration → evaluate on the Stage-5 benchmark → keep
  only if it beats the incumbent on the **held-out blended score with zero regression.** *"Most top
  harnesses are deterministic code, not prompt edits"* — prefer deterministic / robustness mechanisms;
  they transfer across vendors, whereas prompt playbooks are vendor-specific and can backfire (directly
  relevant to a heterogeneous panel).
- **Execution substrate:** a dependency-aware execution graph (bounded escalation retry → patch →
  replan; cross-vendor racing) — adopted as *concepts*, never cited as measured gains.
- **The conductor:** a light router over a frontier pool.

---

## 3. No-regression rollout ordering (additive, flag-gated, shadow-first)

Governed by build-order: **manual-reliable → skill → loop (gate + stop) → then schedule**, and by the
zero-regression rule (one flag, shadow → measured no-regression → enforce, one-flag rollback).

1. **Observability + cost instrument** — thread a tracer through the hot paths; emit
   cost-per-accepted-change + acceptance-rate per loop. Makes everything A/B-testable.
2. **Objective-gate audit (WARN)** — classify each verdict as objective-witness-backed vs judge-only;
   quantify judge-only ships before changing any verdict.
3. **Loop-Library lane specs** — write a one-page spec per recurring lane: objective gate + proof + a
   *bounded* stop condition (streak / cap / clean-exit / convergence).
4. **Loss-function + anti-reward-hacking fences per loop** — state each loop's target / fences /
   instruments / forced-entropy; add a found-nothing-pass canary to every bug-hunt loop.
5. **Cross-vendor check-before-ship gate** — make "a different-vendor check ran" a *recorded, required*
   step (verdict advisory first); confirm each seat answers a low-effort smoke probe before trusting it.
6. **Enforce the objective-gate floor at accept** — only after the WARN cycle quantifies the gap; reuse an
   `UNVERIFIED` verdict, never a silent pass.
7. **Capability-cliff qualification + per-vendor marginal-value** on the hardest tasks; drop rubber-stampers.
8. **Stand up the A/B/C harness benchmark on the private corpus** — the acceptance test.
9. **Memory-as-evolving-layer + shared store mirror**, additive / read-only first.
10. **Automated evolve-the-harness loop, human-in-the-loop first**, gated on the benchmark being trustworthy.

---

## 4. The honest benchmark (restated — it can show you NOT winning)

Superiority is *only* claimed after the Stage-5 benchmark runs on **held-out** data with pinned weights,
showing your heterogeneous truth-gated arm beats single-agent and homogeneous-multi on caught-defect +
agreement soundness at an affordable cost-per-accepted — **and** each panel member earns its seat by
marginal error-catching on the hardest tasks. If the homogeneous arm ≈ yours, or cost ≫ the single-agent
baseline, or false-alarms are high, the benchmark **says so.** Until then, "best-in-class" is a target,
not a result. That is the truth-first position the whole program is built to defend.

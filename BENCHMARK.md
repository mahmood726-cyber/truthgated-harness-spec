# BENCHMARK — the honest benchmark that would PROVE superiority

Companion to [SPEC.md](./SPEC.md) and [WORKFLOW.md](./WORKFLOW.md). This is the acceptance test for the
whole methodology. It is **designed so it can report that you do not win** — that is the point.

**Name (illustrative):** a constrained-budget closed-loop harness benchmark.
**Shape:** a **closed-loop** eval that runs the *same task set* through three harness configurations under
an **identical budget ceiling**, scored on **both correctness/agreement AND efficiency**, on **your
private ground truth**, decided on a **held-out split the agent never reads.**

See [SOURCES.md](./SOURCES.md) for the external designs this is derived from (MADE-style closed-loop eval;
the evolve-the-harness promotion rule and "never read the test split" discipline).

---

## 1. The three arms (the head-to-head)

1. **A — Single-agent baseline.** One vendor, one pass, its own self-check. No cross-vendor, no
   consensus. (The "capable model, brittle scaffold" control.)
2. **B — Homogeneous multi-agent baseline.** N agents, **same vendor/family** (e.g. 3× maker+checkers).
   Tests whether *more agents* alone — without vendor diversity — is the win. This is the honest control
   that isolates the D1 claim: if B ≈ your arm, heterogeneity buys nothing.
3. **C — The heterogeneous truth-gated harness (yours).** Cross-vendor consensus-or-flag (D1) +
   objective-gate floor (D2) + reproduction-or-flag (D3), **all expressed as Conductor-orchestrated
   flows** (D7): the Conductor routes each unit to the strongest live vendor/node, gates against the
   objective floor before accepting any lane's pass, and reroutes on vendor-capacity events. Under the
   same budget.

**All three arms run through the Conductor** (the single control plane); the arms differ only in *what the
Conductor orchestrates* (one vendor / N same-vendor / frontier-heterogeneous panel), so the head-to-head
isolates the panel, not the plumbing.

**Panel-qualification pre-gate (runs before C is even assembled):** each candidate vendor-model must pass
the **capability-cliff qualification** — hold accuracy on the *hardest* subset of your domain, not on easy
tasks. A model that fails the cliff is excluded from C's reasoning/verification seats. A cheaper
same-family verifier candidate is qualified here specifically — kept only if it holds on the hard subset.

---

## 2. The task set (on your corpus)

Drawn from the private corpus, mixing **fix tasks** (a real defect is present — the harness should
catch/repair it) and **regression tasks** (the artifact is correct — the harness must *not* flag it, i.e.
no false alarms). Illustrative task categories:

- **Planted-defect fix tasks with a known answer key** — e.g. an arithmetically-impossible cell in a 2×2
  table (events > N). A "found-nothing pass" fails the canary.
- **Numeric-parity tasks** — reproduction targets checked against a reference implementation to declared
  precision.
- **Discrepancy-detection tasks** — e.g. registry-vs-published pairs.
- **Clean, known-correct artifacts** — regression / no-false-alarm tasks.

**Minimum honest size:** enough that **enumeration doesn't pay** and a single failure mode can't dominate.
(A ~24-task dev set is a *stated* limitation in the literature — treat dev size as a caveat, not a hidden
one.) Split into **dev (tuning)** and **held-out (decision)**; the held-out answer key is *outside* the
agent's tool/file surface.

---

## 3. The exact metrics

Per arm, over the held-out split:

**Correctness / efficacy**
- **caught-defect rate** = fraction of fix tasks whose planted defect was detected (and, where applicable,
  correctly repaired to an objective gate).
- **false-alarm rate** = fraction of regression (correct) tasks the arm wrongly flagged. *Lower is
  better; a harness that flags everything is not "safe", it is useless.*
- **numeric-parity rate** = fraction of reproduction tasks reproduced to declared precision.
- **agreement soundness** = fraction of *agreements* that were actually correct. *Guards the "two
  optimists agree" hack — an agreement on a wrong answer counts against, not for.*

**Per-vendor marginal value (the panel-membership test)**
- **marginal caught-defect** = defects caught *only because* vendor V was on the panel (drop-one
  ablation: remove V, re-score). A member with ~0 marginal catch adds no verification value.
- **rubber-stamp rate** = fraction of V's agreements that were on *wrong* answers. High rubber-stamp =
  negative signal → **V is dropped from the panel.** This directly enforces "frontier-only, non-diluted":
  panel membership is earned on the hardest tasks, not granted by vendor identity.

**Efficiency**
- **cost-per-accepted-change** = total spend / number of accepted-and-correct changes (from real per-run
  cost accounting).
- **wall-clock per accepted change** and **tokens per accepted change** (secondary).

**Control-plane integrity**
- fraction of accepts that carried a Conductor provenance record with an objective witness under them
  (target 1.0); reroute-on-capacity success (a capped vendor's unit completes on a rerouted vendor with no
  lost work).

**Blended promotion score** (decided on held-out only):

```
score = caught_defect_rate
        − λ_fa · false_alarm_rate
        + 0.5 · numeric_parity_rate
        − 0.005 · cost_per_accepted_normalized
```

where `λ_fa` penalizes false alarms and the cost term is normalized so a cheaper arm at equal efficacy
wins. **Weights are design parameters pinned *before* the run and never tuned on the held-out split** —
pinning them is itself a fence.

---

## 4. The win condition (stated so it can fail)

Arm **C is "world-class for its purpose"** iff, on the **held-out** split:

1. **C > A** on caught-defect rate **AND** on agreement soundness (heterogeneous truth-gating catches
   defects a single agent misses), **with false-alarm rate no worse than A**; and
2. **C > B** on caught-defect rate **OR** agreement soundness **at equal-or-lower cost-per-accepted**
   (vendor *diversity*, not just agent count, is doing real work); and
3. C's **cost-per-accepted-change is within a pre-declared multiple** of A's (pinned before the run) —
   i.e. the truth-gating is affordable, not merely correct.

**Honest failure outcomes this design permits (and must report if they occur):**
- If **B ≈ C**, heterogeneity is not paying and D1 is *not* a differentiator — report it, and either
  strengthen the reproduction/gate layers or drop the vendor-diversity claim.
- If **C's cost-per-accepted ≫ A's** without a matching efficacy gain, the harness is correct but not
  *world-class for a working program* — report it and attack D5.
- If **C's false-alarm rate is high**, consensus-or-flag is flagging noise — report it; a harness that
  cries wolf is not defensible.

**Truth-first mandate:** the benchmark is built to be honest. No arm's prompts/config may read the
held-out split; weights and the cost multiple are pinned before running; a "found-nothing pass" on a fix
task is a *failure*, not a pass. Superiority that only appears on the tuning split is not superiority.

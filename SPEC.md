# SPEC — A truth-gated, heterogeneous-vendor verification harness

**Status:** SPEC / north-star blueprint. This is a *methodology* document — a yardstick, not an
implementation. It describes a pattern for building an agent harness that is **best-in-the-world *for a
narrowly-scoped purpose*, provably** — and a benchmark designed so it *could show you not winning*.

> This is a generalised, self-contained blueprint. It captures a methodology so anyone can adopt or
> adapt it. It contains no proprietary code, no private data, and no secrets. Concrete tool names
> (e.g. Claude / GPT / Gemini) appear only as *illustrative examples* of independent model families.

---

## 0. The purpose (anchor — do not drift)

This is **not** a general-purpose agent framework. It is a pattern for a **truth-gated,
heterogeneous-vendor reproduction-and-verification orchestrator** for **quantitative work where the
failure mode is a silently-wrong number, not a crash** — the motivating domain here is
**evidence-synthesis methods research** (meta-analysis, network meta-analysis, diagnostic-test-accuracy,
dose-response, registry-borrowing / transportability) and the data apps built on top of it.

"World-class for its purpose" is a **bounded, testable claim**: best-in-the-world at truth-gated
cross-vendor reproduction-and-verification of quantitative work, *provably* — not "a good agent
framework." Every differentiator below is scoped to that purpose, and the benchmark ([BENCHMARK.md](./BENCHMARK.md))
is designed so it can report that you are *not* winning.

**Defensible edge (the moat):** cross-vendor **consensus-or-flag** verification + **objective
truth-gates** under every accept + a **private eval / ground-truth corpus** that nobody else can score
against. *"The product is a weekend; the eval nobody else can score against is the moat."*

---

## 0.5 Control model — one control plane (the conductor)

**Foundational architectural constraint:** the harness is controlled entirely through **one control
plane** — call it the **Conductor**. There is no second bespoke control daemon; the Conductor *is* the
control plane.

- **The Conductor conducts; it does not do the heavy reasoning.** It occupies the "small manager /
  router" seat: it **routes each unit of work to the right VENDOR** (a frontier model from one of several
  independent families) **on the right NODE** (any machine in your pool), **gates the result, and moves
  on.** It does not itself produce the answer it is gating.
- **Everything runs through the Conductor's primitives.** Drive work lanes (start a task, send a
  message), observe them (read transcripts / list sessions), and run a scheduled **drain-watchdog** that
  reports back so the Conductor can relaunch / refill lanes to capacity. No control logic lives outside
  this loop.
- **Vendor lanes are stateless workers.** All *control* logic — routing, capacity-detection and
  rerouting, truth-gating, consensus-or-flag adjudication, the drain-to-cap loop, and
  cost-per-accepted-change accounting — is expressed as **Conductor-orchestrated flows.** The vendor
  seats hold no control state; the Conductor dispatches to them and owns the gate.
- **The router need not be the biggest model.** A light coordinator routing a pool of frontier models
  can beat any single frontier model (see [SOURCES.md](./SOURCES.md), Trinity). This applies **only to
  the Conductor routing seat** (pure routing / low reasoning) and is a **tested cost lever, not an
  assumption** (§1.5). All *reasoning / reproduction / verification* work runs on frontier models.

This constraint is load-bearing: D1 (consensus-or-flag), D2 (objective-gate floor), D4 (provenance), and
D5 (cost-per-accepted) are all **defined as Conductor flows**, and D7 makes the control plane itself a
differentiator.

---

## 1. The seven differentiators (what makes it best-in-class FOR THIS PURPOSE)

Each differentiator states **the claim**, **why it is defensible for this purpose**, and **the objective
signal that proves it holds** (so none is a vibe).

### D1 — Cross-vendor consensus-or-flag as a first-class primitive
- **Claim:** Every ship-eligible verdict is produced by **≥2 independent model families** (e.g.
  Anthropic / OpenAI / Google), and disagreement **flags** rather than silently resolving to the
  optimistic side. Homogeneous agreement is explicitly *not* counted as independent corroboration.
- **Why defensible:** Correlated judges share failure modes ("many judges, few effective votes"). A
  single-vendor stack — however strong — cannot decorrelate its own blind spots. Heterogeneous seats are
  a *structural*, not prompt-level, advantage, and robustness of this kind *transfers across models*
  (unlike prompt playbooks).
- **Proof signal:** a "distinct families" invariant holds in 100% of ship verdicts; a
  `cross_vendor_check_present(vendor=…)` field on every verdict; the benchmark's heterogeneous arm beats
  the homogeneous arm on caught-defect rate at equal-or-lower cost.

### D2 — An objective truth-gate floor under every accept
- **Claim:** No ship decision rests on **judges or consensus agreeing alone.** Under every accept there
  is a **named objective witness** — a test-suite exit code, a `--check` parse, a build, a runtime
  browser check, a byte/numeric diff, numerical-continuity, or a reference-implementation parity check —
  or the task is explicitly reclassified **human-in-the-chair** (not loop-eligible).
- **Why defensible:** This is the load-bearing loop-engineering rule: *"VERIFY = an objective gate, not a
  second agent agreeing."* It is what makes consensus-or-flag *sound* — consensus is only as strong as
  the objective witness beneath it. It fences the reward-hack where two optimists agree on a wrong answer.
- **Proof signal:** the objective-gate audit reports **0 ships resting on consensus-without-a-floor**
  once promoted (starts as a measured WARN count).

### D3 — Reproduction-or-flag discipline (independent re-derivation, not review)
- **Claim:** A quantitative result is accepted only when an **independent vendor reproduces the number to
  declared precision** (byte/numeric parity), or the divergence is flagged. Verification is
  *re-derivation*, not "a reviewer read it and it looked right."
- **Why defensible:** For this class of work the failure mode is a *silently wrong number*, not a crash.
  Only independent reproduction catches it. This is also the exact shape of the private-corpus moat:
  ground-truth pairs are reproduction targets, not opinion targets.
- **Proof signal:** every accepted quantitative claim carries ≥1 independent-reproduction witness id; the
  benchmark measures caught planted-numeric-defects.

### D4 — Full auditability / provenance of every claim
- **Claim:** Every verdict is reconstructable end-to-end: which witnesses ran, which vendor produced
  what, span-level timing/tokens, a signed result bundle, and a structured (JSONL) finding stream — with
  freshness / replay protection so a stale pass cannot be re-presented as current.
- **Why defensible:** A claim you cannot audit is not a claim. Provenance is what lets a reviewer (or a
  future session) trust an accept without re-running it, and is a precondition for the moat to compound.
- **Proof signal:** span coverage = 1.0 over the expected pipeline stages on a traced verdict; the signed
  bundle verifies; an explicit `UNVERIFIED` verdict is emitted (never a silent pass) when a witness SKIPs
  on a missing baseline.

### D5 — Cost-per-accepted-change economics made visible (and fenced)
- **Claim:** The harness measures **cost-per-accepted-change** and **acceptance rate** per loop from real
  spend, surfaces loops below a practitioner break-even (labelled a *heuristic*, not a measured
  constant), and enforces a hard money / wall-clock ceiling.
- **Why defensible:** A truth-gated harness that is 10× the cost of a single agent for the same accepted
  output is not world-class *for a working program* — it is a science project. Efficiency is half the win
  condition on purpose. Making spend visible also fences the "burn tokens to look busy" reward-hack.
- **Proof signal:** a cost-per-accepted-change number exists per loop; the benchmark scores efficiency as
  a first-class axis, not a footnote.

### D6 — A private eval / ground-truth moat that grows with use
- **Claim:** Superiority is scored against a **private, curated, truth-gated corpus** — for example
  *registry-vs-published discrepancy pairs*, a curated gold reference library, and
  reference-implementation parity fixtures — that is **blinded to the agent at run time** (never readable
  mid-run) and **grows** as production failures are auto-promoted into regression fixtures.
- **Why defensible:** Anyone can clone a dashboard generator in a weekend; nobody else can score against
  your corpus. The moat is the eval, and it compounds: every caught failure becomes permanent coverage.
- **Proof signal:** the benchmark runs *on this corpus*; the held-out split is never read during a run;
  fixture count grows monotonically as failures are classified.
- **How to adapt (pattern, not data):** pick a ground-truth source *you* can build that a competitor
  cannot cheaply reproduce — the pattern is "maintain a private eval a competitor can't score against."
  Keep the actual answer keys out of the agent's file/tool surface. *This blueprint deliberately ships
  no answer keys, fixtures, or curated data — only the pattern.*

### D7 — One control plane (the Conductor)
- **Claim:** All control is exercised through **one** plane — the Conductor — which routes each unit of
  work to the strongest live vendor on the right node, gates the result against the objective-gate floor
  (D2) before accepting any lane's "pass", reroutes on vendor-capacity events, and records provenance
  (D4) of every claim through the flow. There is no second control path.
- **Why defensible:** A single, uniform control plane is what makes the other six *auditable and
  evolvable*: every accept, reroute, and cost event passes through one instrumented conductor, so the
  whole system is one A/B-testable loop rather than a tangle of bespoke daemons. It is also the
  validated shape (a light conductor over a frontier pool beats any single model), and it means the
  control logic can be improved without touching the vendor workers.
- **Proof signal:** every ship verdict carries a Conductor provenance record (vendor, node, gate outcome,
  cost); a vendor-capacity event produces a recorded reroute to a non-capped vendor with no lost work; no
  "pass" is accepted without a named objective witness under it (D2).

**Why exactly these seven:** D1–D3 are the *correctness* core (independent, gated, reproduced); D4 is the
*trust* layer (auditable); D5 is the *economics* that keep it usable; D6 is the *moat* that makes the
whole thing defensible and measurable; D7 is the *control plane* that binds the other six into one
instrumented, evolvable conductor. Drop any one and the claim "best-in-the-world for its purpose" stops
being provable.

---

## 1.5 Vendor panel / model selection — FRONTIER-heterogeneity, qualified on the HARDEST tasks

**Principle (empirical):** the cross-vendor panel must be **heterogeneity across FRONTIER models.**
Weaker / cheaper models are **excluded from reasoning / reproduction / verification seats.** This is not
cost-indifference — for this work a weak model in a verification seat adds *correlated, obvious* failures
(little independent signal) and can *rubber-stamp* a wrong number, which is worse than no panel member at
all.

### 1.5.1 The capability-cliff evaluation principle (the load-bearing rule)
Observed failure mode: **weaker models often perform *well on moderate tasks* but *collapse on really
complex, long-horizon* reasoning/code.** The cliff is **invisible on easy tasks.** Therefore:

1. **A model's fitness for a seat MUST be judged on your HARDEST tasks, never on easy/average ones.**
   Demo / standard-benchmark performance is **not sufficient** — the cliff won't show there. The
   panel-qualification eval stresses each candidate on the *most complex* tasks in your domain (e.g.
   multi-arm NMA parity, DTA bivariate convergence, a hard k-fold, long agentic bug-hunts). A candidate
   qualifies for a seat only if it holds accuracy **there.**
2. **Frontier-heterogeneity matters because frontier models from different vendors fail *differently*** —
   independent, subtle failures → genuine error-catching when they disagree. Weak models fail in
   *correlated / obvious* ways → little independent signal. Decorrelation is only valuable between models
   each individually *above* the cliff.
3. **"Push frontier models hard on the hardest tasks; do not dilute the panel with weak models to save
   cost."**

### 1.5.2 Per-seat model selection (recommendations are TESTED, not assumed)
Recommend a model per seat only **after** (or with a concrete plan to) empirically qualify it on your
hardest tasks per §1.5.1.

| Seat | Role | Model policy | Evaluation to run |
|---|---|---|---|
| **Worker + verifier (family A)** | reasoning / method dev / verification | top-tier for work | test a cheaper same-family model for the *verifier* role — acceptable **only if** it holds accuracy on the hardest tasks; qualify on the cliff, don't assume |
| **Worker + verifier (family B)** | strongest bug-finder / cross-vendor check | strongest frontier model available | re-evaluate as newer frontier models land; never drop below frontier for verification |
| **Worker + verifier (family C)** | third-family decorrelation seat | strongest frontier model available | qualify on hard tasks |
| **Conductor (routing only)** | route / gate / move-on, low reasoning | frontier by default; a cheaper model is acceptable **only here** | tested option, gated on not hurting outcomes — measure that routing quality and end-to-end outcomes do not regress before adopting a cheaper conductor |

**Reconciliation:** the "router needn't be the biggest model" result applies **only to the narrow
Conductor routing seat.** Every actual reasoning / reproduction / verification unit runs on a **frontier**
model. A cheaper conductor is a *tested* cost lever, never a default.

### 1.5.3 Benchmark implication — measure each vendor-model's MARGINAL value
The consensus-or-flag eval must **measure the marginal verification value of each panel member**: a
vendor-model that mostly errors, or mostly rubber-stamps (agrees without catching planted defects),
**adds no verification value and is dropped from the panel.** Panel membership is *earned* by genuinely
catching errors on the hardest tasks, not granted by vendor identity.

---

## 2. Non-goals (explicit — prevents drift)

- **Not** a general agent framework, IDE, or chatbot. No feature earns its place unless it serves
  truth-gated reproduction-and-verification.
- **Not** "most agents" or "most autonomy" — a homogeneous-multi-agent arm exists precisely to refuse the
  "more agents = better" story unless measured.
- **Not** a novelty collector. New techniques are candidate *mechanisms*, adopted only if the benchmark
  shows efficacy/cost improvement with zero regression.
- **Not** self-mutating over production on day one. Human-in-the-loop promotion; an autonomous proposer is
  a north-star gated on the evaluator being trustworthy.
- **Not** a claim of superiority you cannot currently prove. Until the benchmark runs on held-out data,
  "world-class" is a *target*, not a stated result.
- **Not** a second control plane. All control runs through the Conductor (§0.5 / D7).
- **Not** a weak-model panel. Reasoning / reproduction / verification seats are **frontier-only** (§1.5).

---

## 3. Governance (how a change earns "world-class")

Every harness change is: **additive / behind a flag / shadow-mode first** → measured **zero-regression**
shadow phase → enforce, with **one-flag rollback** and the prior code path intact for one full cycle. A
change is *promoted* only when it beats the incumbent on the benchmark's **blended score on held-out
evals.** Build-order discipline governs sequencing: **manual-reliable → skill → loop (gate + stop) → then
schedule.** Single-writer-per-repo; no force-push; gated commits.

*This spec is the yardstick. [WORKFLOW.md](./WORKFLOW.md) maps the end-to-end pipeline as
Conductor-orchestrated, loop-engineered stages; [BENCHMARK.md](./BENCHMARK.md) is the acceptance test;
[SOURCES.md](./SOURCES.md) attributes every external idea.*

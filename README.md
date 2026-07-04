# truthgated-harness-spec

**A starter spec for a truth-gated, heterogeneous-vendor verification harness.**

This is a **methodology blueprint** — a self-contained, adopt-or-adapt spec for building an AI-agent
harness that is *best-in-the-world for a narrowly-scoped purpose, provably*, with a benchmark **designed
so it can show you not winning.** It is a spec, not an implementation: there is no proprietary code, no
private data, and no secrets here — only the pattern.

The motivating purpose is **truth-gated cross-vendor reproduction-and-verification of quantitative
evidence-synthesis work** (meta-analysis, NMA, DTA, dose-response, registry-borrowing) — the domain where
the failure mode is a *silently-wrong number*, not a crash. The pattern generalises to any domain with
that property.

---

## What's here

| File | What it is |
|---|---|
| **[SPEC.md](./SPEC.md)** | The core spec: the purpose anchor, the single-control-plane model (the "Conductor"), the **seven differentiators** (each with an objective proof-signal), and frontier-heterogeneity model selection with the **capability-cliff** rule. |
| **[WORKFLOW.md](./WORKFLOW.md)** | The end-to-end **6-stage pipeline** (problem → private-eval moat → method dev → cross-vendor verification → benchmark → publish), each stage as a loop with a gate + stop + fence, plus the two cross-cutting layers (evolving memory; evolve-the-harness). |
| **[BENCHMARK.md](./BENCHMARK.md)** | The **honest A/B/C benchmark**: single-agent vs homogeneous-multi vs heterogeneous truth-gated, constrained-budget closed-loop, scored on correctness **and** cost-per-accepted-change, decided on a held-out split — with the win condition *and the honest failure outcomes* stated up front. |
| **[docs/LOOP-ENGINEERING.md](./docs/LOOP-ENGINEERING.md)** | The anatomy of a single loop that can't cheat you: objective gate · proof artifact · bounded stop · external state · **fenced 4-part loss function**, plus the anti-reward-hacking rule and build-order discipline. |
| **[SOURCES.md](./SOURCES.md)** | Attribution for every external idea (evolve-the-harness, Loop Library, MADE, Trinity, Loss-Function-Development, RLVR+human-demos, Claude Code docs, AutoMem/MemClaw, Ragas, skills collections), with URLs and honesty caveats. |

---

## The core ideas in one screen

1. **One control plane — the Conductor.** A light router assigns each unit of work to the strongest live
   *frontier* vendor on the right node, gates the result, and moves on. It does not do the heavy
   reasoning. The router need not be the biggest model (a *tested* cost lever, not an assumption).
2. **Cross-vendor consensus-or-flag.** Every ship verdict is produced by ≥2 *independent model families*;
   disagreement **flags** instead of silently resolving to the optimistic side. Homogeneous agreement is
   not independent corroboration.
3. **An objective-gate floor under every accept.** No ship rests on judges agreeing alone — there is
   always a named mechanical witness (test / diff / build / reference-parity / runtime check), or the task
   is reclassified human-in-the-chair.
4. **Reproduction-or-flag.** A number is accepted only when an independent vendor *re-derives* it to
   declared precision — verification is re-derivation, not "a reviewer read it."
5. **A private-eval moat (as a pattern).** Score yourself against a private ground-truth corpus a
   competitor can't reproduce (e.g. registry-vs-published gaps), blinded at run time, growing as failures
   become fixtures. *"The product is a weekend; the eval nobody else can score against is the moat."* —
   **keep your actual ground truth private; publish only the pattern.**
6. **Cost-per-accepted-change, made visible and fenced.** A correct harness that costs 10× a single agent
   for the same output is a science project, not a working program. Efficiency is half the win condition.
7. **Frontier-heterogeneity, qualified on the HARDEST tasks.** Weak models collapse on complex,
   long-horizon work — and the cliff is invisible on easy tasks. Qualify every seat on your hardest tasks;
   drop any panel member that rubber-stamps.
8. **The harness evolves, benchmark-gated.** Propose one mechanism → evaluate on the held-out benchmark →
   keep only on a zero-regression win. Prefer deterministic mechanisms; they transfer across vendors.

---

## How to use it

- **Read it as a yardstick.** SPEC.md is a north-star contract: measure a real harness against the seven
  differentiators and their proof-signals. If a differentiator has no objective signal, it's a vibe.
- **Adopt the discipline before the automation.** Build-order is *manual-reliable → skill → loop (gate +
  stop) → then schedule.* Don't schedule a loop that has no objective gate or no bounded stop.
- **Stand up the benchmark early.** It's the acceptance test for everything else. Pin the weights and the
  cost multiple *before* the run; decide on the held-out split only.

## How to adapt it

- **Swap the domain.** The pattern fits any domain where the failure mode is a silently-wrong result, not
  a crash. Replace the evidence-synthesis examples with your own hardest tasks.
- **Swap the vendors.** "Family A / B / C" are placeholders for any ≥2 *independent* frontier model
  families. The differentiator is *independence*, not any specific product.
- **Build your own moat.** The one part you should **not** copy is a dataset — there isn't one here.
  Pick a private ground-truth source *you* can build and a competitor cannot cheaply reproduce, and keep
  its answer keys out of the agent's file/tool surface.

---

## Provenance & honesty

This blueprint synthesizes public work by others (see [SOURCES.md](./SOURCES.md)) with a practitioner
workflow. It is deliberately a **spec**, not a codebase: implementation details, private ground-truth
data, infrastructure specifics, and secrets are intentionally excluded. Where a cited source's own numbers
were anecdotal, that caveat is carried through — please don't cite them as measured constants. Verify
external references before relying on them.

## License

[MIT](./LICENSE). Use it, fork it, adapt it. Attribution appreciated but not required.

# SOURCES — external ideas this blueprint builds on

This methodology synthesizes public work by others. Each idea below is attributed to its source. Where a
source's own numbers were anecdotal or unverified in the original, that caveat is carried through here —
do not cite them as measured constants.

> Attribution note: URLs and identifiers are given as published by their authors. Verify them yourself
> before citing; some may move. Nothing in this repo is endorsed by these authors.

| Idea used here | Source | Reference |
|---|---|---|
| **Evolve the harness, not the model** — an automated, benchmark-gated loop that mutates the *harness* (prompts, tool wiring, retrieval, verification, routing, completion criteria) and keeps a change only if it beats the incumbent on a blended efficacy/cost score with zero regression, decided on held-out data. "One mechanism per iteration", "never read the test split", "most top harnesses are deterministic code, not prompt edits." | Joel Niklaus, *"Don't Train the Model, Evolve the Harness"* | https://huggingface.co/spaces/joelniklaus/harness-optimization · related reference impl: https://yoonholee.com/meta-harness/ , https://github.com/stanford-iris-lab/meta-harness |
| **Loop Library** — recurring agent loops each stated as *objective + proof + explicit stop condition* (multi-LLM convergence, adversarial-review, quality-streak, error-sweep, research-to-artifact, post-release baseline). Supplies bounded stop conditions for each lane. | Forward Future — *Loop Library* | https://signals.forwardfuture.com/loop-library/ |
| **Constrained-budget closed-loop benchmark (MADE-style)** — composable maker/checker/verifier components, ablation arms, efficacy + cost-per-accepted-change vs a single-agent baseline. The shape of the A/B/C benchmark. | MADE benchmark | arXiv:2601.20996 |
| **A tiny coordinator routing a frontier pool beats any single model** — tri-role (thinker/worker/verifier) delegation, loop until the verifier ACCEPTs; the router need not be the biggest model. Grounds the Conductor + mandatory cross-vendor check. | TRINITY — *An Evolved LLM Coordinator* (Sakana AI, ICLR 2026) | arXiv:2512.04695 · https://sakana.ai/trinity/ |
| **Loss-Function Development (LFD)** — how to design the loss a long-running agent loop descends and *fence it against reward-hacking*: 4-part loss (target / constraints / instruments / forced-entropy); *"every cheap path you don't fence off is a direction the optimizer sprints down"*; the "eval nobody else can score against is the moat" framing. (The 50× / 30-hour / $40 figures are the author's anecdotes, not measured constants.) | Elvis Sun — *Loss-Function Development* / `/lfd-design` | https://github.com/elvisun/loss-function-development |
| **A single verifiable reward gets hacked; a second, harder-to-game signal nearly eliminates it** (RLVR + human demonstrations). Grounds "the cross-vendor check is the harder-to-game complement to the objective scalar." | "Right in the Right Way" (RLVR + human-demos, MIT) | See publisher; verify before citing |
| **Claude Code primitives** — subagents (checker), hooks (gates/constraints), headless mode (cost instrument), skills/slash-commands (loops), MCP (connectors). The build substrate every mechanism is grounded in. | Anthropic — Claude Code docs | https://code.claude.com/docs |
| **Memory as a learnable, evolvable skill** distilled from your own trajectories (when/what to store, how to retrieve) — the framing that the memory layer should not sit static. | AutoMem | arXiv:2607.01224 |
| **Machine-readable shared memory store** — a concrete tool for the shared-signal bus; mirror stuck-failures / progress / circuit-state files into it additively with contradiction-detection. | MemClaw (Caura, Apache-2.0) | See project repo; verify before citing |
| **Extraction-fidelity evaluation** — LLM-judged faithfulness of an extracted value against its source, used only as a WARN-tier complement to deterministic gates. | Ragas | https://github.com/explodinggradients/ragas |
| **Reusable skill patterns** — TDD red-green-refactor inner loop, disciplined bug-diagnosis loop, two-axis (standards + spec) code review, PRD/context authoring, triage state machine. Starting shapes for lane skills. | Curated agent-skill collections (e.g. mattpocock/skills) | https://github.com/mattpocock/skills |

---

## How the sources compose

- **The meta-loop** (Niklaus) is the outer control structure: propose one harness mechanism → evaluate →
  keep only on a zero-regression held-out win.
- **The benchmark it scores against** (MADE-style) is the evaluator — the constrained-budget closed-loop
  A/B/C eval.
- **The candidate mechanisms** are the individual loops (Loop Library templates + your own lanes), each
  with an objective gate, a proof artifact, and a bounded stop condition.
- **The empirical validation of the shape** (Trinity) is why a light conductor over a frontier pool, with
  an explicit accept step, is the right architecture.
- **The loss-design discipline** (LFD, reinforced by RLVR+human-demos) is how each loop's loss is
  designed and fenced against reward-hacking.
- **The build substrate** (Claude Code primitives) is what each mechanism is actually built from.
- **The memory layer** (AutoMem framing + a machine-readable store) is the cross-cutting evolving state.

# PlanProof

<!-- Add a hero/banner image here. -->

**Provable plan review for [Claude Code](https://claude.com/claude-code).** Most tools send your plan to an LLM and hand back notes. PlanProof measures your plan against a completeness rubric, provably tightens it, and refuses to claim a fix it can’t verify.

*The name is the goal: a plan made as close to bulletproof as review, measurement, and a verified improvement loop can get.*

> **Status: Preview — building in public. Maturity: Draft.** Nothing here is implemented beyond the review pass (see Roadmap); this README doubles as the project plan. At **Draft** tier, deferred fields (license, exact metric targets and dates) are allowed — they must resolve before this plan is marked **Execution-ready**.

## The problem

Pasting a plan into an LLM gives you prose notes that change every run, measure nothing, and sound just as confident when they’re wrong. There’s no score, no proof that a suggested change actually made the plan better, and nothing stopping the model from “improving” a doc by inserting hollow “TBD” placeholders that read like progress. The review is a vibe.

## What PlanProof does

A three-stage pass, each stage leading with the one line that survives skimming:

1. **Review** — runs a senior reviewer’s lenses over the plan and returns one scannable review: the top finding plus one line per lens, each free to say “no issue.” *(Built — see the [`plan-proof`](plan-proof/SKILL.md) skill.)*
1. **Measure** — scores the plan against a *deterministic* completeness rubric. On a template-conformant plan, the same plan yields the same score every time; free-form plans are scored best-effort (see [Determinism](#determinism)). *(Planned.)*
1. **Improve** — runs a bounded loop that proposes edits, re-scores against the rubric, and keeps a change only if the score rises with no check regressing. Returns a verified, numbered gain — or a clean “no gain found.” *(Planned.)*

## What makes it different

|             |Paste-to-LLM                   |PlanProof                                                           |
|-------------|-------------------------------|--------------------------------------------------------------------|
|What you get |Prose notes, different each run|A scored review against a fixed rubric                              |
|Measurement  |None (“looks good to me”)      |Deterministic completeness score                                    |
|Improvement  |Suggestions you apply by hand  |A loop that keeps only changes that raise the score                 |
|Honesty      |Confident even when wrong      |Refuses a fix it can’t verify; reports “no gain” plainly            |
|Repeatability|Non-deterministic              |Deterministic on template-conformant plans; best-effort on free-form|

The wedge in one line: *auto-improve isn’t a feature name here — it’s the proof the feedback is real.*

## How it works

The engine composes proven primitives rather than inventing new ones: the five **review lenses** come from [Giant Brains Claude Skills](#relationship-to-giant-brains-claude-skills), and the **improvement loop** is `auto-improve`’s mutate -> re-score -> keep-or-revert search, bound here to a deterministic plan rubric instead of a code oracle. The editor and the grader are strictly separated, so the loop can never grade its own homework — the single failure that would discredit the whole idea.

That guarantee is **scoped**: it holds on structured (template-conformant) input, where the checks are unambiguous. Free-form plans are parsed first and scored best-effort — and PlanProof labels which path it took.

## The completeness rubric (the oracle)

The load-bearing, novel piece. A plan is not free prose — it’s a structured artifact with checkable invariants, so it *can* have an un-gameable oracle where arbitrary prose can’t. Each check emits PASS or FAIL; the score is rubric coverage.

|Check                      |Rule                                                                                                             |Fails when                                                         |
|---------------------------|-----------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------|
|Measurable success criteria|Every objective states a target with a number, unit, and date                                                    |Missing, “TBD”/“N-A”, or vague (“improve”, “faster”) with no figure|
|Rollback for one-way doors |Every irreversible step (delete / drop / migrate / deploy / publish, or explicitly flagged) has a paired rollback|No contingency stated                                              |
|No orphan tasks            |Every task maps to at least one objective                                                                        |A task has no parent objective                                     |
|No empty objectives        |Every objective has at least one task                                                                            |An objective has zero tasks                                        |
|Risks have mitigations     |Every named risk has a stated mitigation or owner                                                                |A bare risk with no response                                       |
|Explicit dependencies      |Cross-task dependencies are named, not implied by ordering words                                                 |“after”/“once” with no named dependency                            |
|No placeholders            |Required sections contain no TODO / TBD / ??? / lorem / empty fields                                             |Any placeholder in a required field (subject to maturity tier)     |

**Anti-gaming, by design:** placeholders, “TBD”, and unreferenced criteria *fail* — so the loop can’t win points by inserting empty scaffolding. Missing information fails; it is never fabricated.

### Determinism

The “deterministic / same score every time” guarantee holds for **template-conformant** plans, where structure is unambiguous. For free-form markdown, PlanProof extracts structure first — and extraction is not deterministic — so free-form scores are **best-effort** and labeled as such. The provable claim is scoped to structured input, not asserted over arbitrary prose. (This is why the schema decision gates the rubric spec.)

### Maturity tiers

A preview legitimately defers a license and final dates; flagging it for those would be crying wolf — and “honest signal, not constant alarm” is a core principle. So the rubric runs in tiers:

- **Draft** — suppresses deferred-field checks (license, exact targets/dates). Surfaces only structural gaps.
- **Execution-ready** — enforces every check. This is the bar for a plan you’re about to run, and the bar this repo’s own planning docs must clear in CI.

(What forces the Draft -> Execution-ready transition is itself an open decision — see [Open decisions](#open-decisions).)

## The lenses

Reused near-free: these are prompt-only and artifact-agnostic, so they repoint at a plan doc with zero code change.

|Lens (from Giant Brains)|The review question on a plan                                     |What it catches                           |
|------------------------|------------------------------------------------------------------|------------------------------------------|
|take-a-step-back        |Is this the right problem, and what’s the most fragile assumption?|Plans that solve the wrong thing          |
|iron-triangle           |What does this plan trade away — scope, time, or quality?         |Unpriced tradeoffs                        |
|blast-radius            |How big and how reversible are the changes it proposes?           |Underestimated scope, hidden one-way doors|
|baseline-spec           |Is success defined measurably?                                    |Goals you can’t tell you’ve met           |
|bottom-line             |When the doc balloons — what’s the actual call?                   |Analysis paralysis                        |

`auto-improve` is deliberately *not* a lens — it’s the improvement engine, and it runs against the deterministic rubric, never against free prose.

## Architecture

- **Orchestrator (`plan-proof`)** — runs the lenses and aggregates into one scannable review; each lens may report “no issue.” *(Built.)*
- **Completeness checker (the oracle)** — deterministic, scriptable; emits per-check PASS/FAIL plus a machine-readable score the loop consumes. The moat. *(Planned.)*
- **Improve loop** — `auto-improve` bound to the checker’s score: short budget, circuit breaker, keep-or-revert, honest “no gain” output. Editor and grader strictly separated. *(Planned.)*
- **Recommended plan template** — the on-ramp, and the upsell to the strongest version of the oracle. *(Planned.)*
- **Lens sourcing** — the five lenses are pulled from Giant Brains read-only (git submodule or pinned vendor + sync script). Never fork-and-drift. *(Planned; currently inlined in the `plan-proof` skill for the slice.)*

## Success criteria

How PlanProof knows *it* works (baseline-spec, applied to itself). Provisional targets; exact corpus and dates set at Execution-ready:

- [ ] **Determinism** — a template-conformant plan scored twice returns identical scores (100% reproducibility).
- [ ] **No regressions** — the improve loop lands zero edits that lower the score or break a previously-passing check.
- [ ] **Real lift** — on a fixed corpus of >=20 known-weak plans, median completeness rises by >=25 points after the loop.
- [ ] **Honest no-op** — on already-tight plans, the loop reports “no gain found” with zero fabricated or placeholder edits.

## Risks & mitigations

|Risk                                                                 |Mitigation                                                                                                                                                                  |
|---------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|The improve loop games the oracle                                    |Deterministic checker; editor and grader strictly separated; placeholders fail by design                                                                                    |
|Non-deterministic extraction undermines “provable” on free-form input|Scope the guarantee to template-conformant plans; label free-form scores best-effort                                                                                        |
|Rubric over-flags early/preview docs (crying wolf)                   |Maturity tiers — Draft suppresses deferred-field checks                                                                                                                     |
|Lenses drift from Giant Brains                                       |Read-only sourcing (submodule or pinned vendor + sync); single source of truth                                                                                              |
|Repo split fragments the Giant Brains audience                       |PlanProof is additive (app on primitives), not a fork; Giant Brains stays intact and un-relicensed (rollback in [Relationship](#relationship-to-giant-brains-claude-skills))|

## Roadmap

Each phase ships behind an explicit **Exit** gate — the verifiable condition that says it’s done.

### Phase 0 — Scaffold

- [ ] Create repo, choose license (default: match Giant Brains), land this README as the plan
- [ ] **Lock the schema decision** (schema-light + scoped guarantee vs schema-enforced) — it gates the rubric spec
- [ ] Decide lens-sourcing method and wire it (submodule, or pinned vendor + sync)
- [ ] Pull in the five lenses from Giant Brains, read-only

**Exit:** repo builds, the five lenses import read-only, and the schema decision is recorded as resolved in Open decisions.

### Phase 1 — Review (quick win, already beats paste-to-LLM)

- [x] Orchestrator skill (`plan-proof`): run the five lenses, aggregate into one review
- [x] Output shape: top finding + one line per lens, each can report “no issue”
- [x] Repoint each lens’s framing at a plan / decision doc (prompt-only, no code)

**Built:** the [`plan-proof`](plan-proof/SKILL.md) skill plus a worked example ([`examples/sample-plan.md`](examples/sample-plan.md) and its [review](examples/sample-plan-review.md)). The lenses are inlined for the slice; sourcing them read-only from Giant Brains is still the open Phase 0 task.

**Exit:** the orchestrator returns one aggregated review (top finding + one line per lens) on a sample plan, and a clean plan produces zero false escalations. *(Demonstrated: the sample review and the skill’s clean-plan example.)*

### Phase 2 — Measure (the moat)

- [ ] Deterministic completeness checker: per-check PASS/FAIL + score
- [ ] Placeholders / TODO / TBD / N-A explicitly FAIL
- [ ] Orphan-task and empty-objective detection
- [ ] Rollback-for-one-way-door detection
- [ ] **Maturity tiers** — Draft suppresses deferred-field checks; Execution-ready enforces all
- [ ] Emit a machine-readable score the loop can consume
- [ ] **CI gate** — this repo’s own planning docs must pass the rubric at Execution-ready

**Exit:** the checker scores a sample plan identically across two runs, placeholders fail, and the CI gate runs against this repo’s own planning docs.

### Phase 3 — Improve (the hook)

- [ ] Bind `auto-improve`‘s loop to the checker (depends on Phase 2’s machine-readable score)
- [ ] Enforce strict editor/grader separation (no self-grading)
- [ ] Short budget + circuit breaker; revert any edit that doesn’t raise the score
- [ ] Honest “no gain found” output when the plan is already tight

**Exit:** the loop raises the score on a known-weak plan with zero regressions, and returns “no gain found” on an already-tight plan.

### Phase 4 — On-ramp and polish

- [ ] Recommended plan template (on-ramp + strong-oracle upsell)
- [ ] More worked examples: weak plans before/after, with scores
- [ ] Install docs for Claude Code and web/desktop
- [ ] Banner image

**Exit:** template and before/after examples are published, and install docs are verified on Claude Code and web/desktop.

**Critical path:** schema decision (P0) -> rubric spec (P2) -> improve loop (P3, which consumes the machine-readable score).

## Open decisions

- [ ] **Schema-light vs schema-enforced** — *Recommendation:* ship schema-light for reach, but scope the deterministic (“provable”) guarantee to template-conformant plans; free-form input is scored best-effort and labeled. **Lock before the rubric spec (Phase 0)** — it shapes every check. *Reversible:* a decision, not yet code.
- [ ] **Maturity-tier enforcement** — what forces a plan from Draft to Execution-ready, and stops permanent-Draft from suppressing checks forever? Options: a plan declared “ready to execute” must score at Execution-ready, or the tier is set by an external signal rather than the doc itself. *Open — left unowned, it’s a self-grading escape (the same failure mode the rubric exists to close).*
- [ ] **Template vs free-form distribution** — the “provable” guarantee holds on template-conformant plans, but the on-ramp invites free-form, so the headline may apply to a minority of real use. Decide whether to price this up front (label free-form best-effort prominently) or push users toward the template harder. *Open.*
- [ ] **Lens sourcing** — git submodule vs pinned vendor + sync script. Constraint: one source of truth, never fork-and-drift. *Rollback:* pin a ref so an upstream change can’t silently break us; vendoring is the exit if submodules chafe.
- [ ] **v1 scope** — ship Review-only first, or Review + Measure + Improve together?
- [ ] **License** — default to matching Giant Brains (GPL v2.0) to avoid compatibility surprises; deferred-allowed at Draft tier. *One-way once contributors land* — decide before accepting outside PRs.

*Decided: product name is **PlanProof** (mirrors “bulletproof”); the entry skill is `plan-proof`.*

## Design principles

Carried over from Giant Brains, applied to documents:

- **Make the implicit explicit** — success criteria, rollbacks, and dependencies stated, not implied.
- **Lead with the line that survives skimming.**
- **Refuse rather than fake it** — the loop rejects any gain it can’t verify; placeholders fail by design.
- **Determinism is scoped, not assumed** — the guarantee holds on structured input; free-form is best-effort and labeled.
- **One reversibility vocabulary** — Easy / Costly / One-way door.
- **Honest signal, not constant alarm** — lenses stay quiet where there’s no issue, and maturity tiers keep a preview from being flagged for fields it’s allowed to defer.
- **Deterministic oracle, separated grader** — the improver never grades its own edits.

## Relationship to Giant Brains Claude Skills

PlanProof is the **application**; [Giant Brains Claude Skills](https://github.com/) is the **primitives**. The five review lenses live there and are sourced here read-only so a single source of truth never forks. `auto-improve`’s loop is reused as the improvement engine, bound here to a deterministic plan rubric rather than a code oracle. Giant Brains stays a general decision-hygiene toolkit; PlanProof is one job built on top of it — provable plan review.

**Rollback for the split itself** — its biggest one-way door: if PlanProof doesn’t earn traction, archive the repo and fold the `plan-proof` orchestrator back into Giant Brains as an example. This stays cheap precisely because standing PlanProof up deletes nothing from Giant Brains and never relicenses it — the split is additive, so unwinding it costs an archive and a move, not a migration.

## Non-goals

- **Not a free-prose improver** — arbitrary prose has no un-gameable oracle, so an improvement loop would game itself. Out of scope by design.
- **Not a replacement for human sign-off** — it tightens and scores; you still decide.
- **No silent edits** — every change is shown with its score delta before it lands.
- **No invented facts to pass a check** — missing information fails; it is not fabricated.

## Proposed layout

```
.
├── plan-proof/SKILL.md          # entry skill: runs lenses -> one review (BUILT)
├── rubric/                      # the deterministic oracle, the moat (planned)
│   ├── SKILL.md
│   └── checks/                  # per-check PASS/FAIL logic
├── improve/SKILL.md             # auto-improve loop bound to the rubric (planned)
├── lenses/                      # sourced read-only from Giant Brains (planned)
├── templates/plan.md            # recommended plan template, the on-ramp (planned)
├── examples/
│   ├── sample-plan.md           # BUILT
│   └── sample-plan-review.md    # BUILT
└── README.md
```

## Maintainer & reviewers

- **Maintainer:** Noel Saw
- **Built with:** Claude Code (Effort Max)

## Sponsored by

This project is supported by two Southern California meetup communities and HiQS.ai.

- [Claude & AI Tools — Ventura County](https://www.meetup.com/claude-ai-tools-ventura-county/)
- [Love2SoCal — Vibe Coding Meetup](https://www.meetup.com/love2socal/)

## License

TBD — see Open decisions; deferred-allowed at Draft tier. Default is to match Giant Brains Claude Skills (GPL v2.0).

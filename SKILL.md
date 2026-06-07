-----

## name: plan-proof
description: Review a project plan, decision doc, spec, RFC, design proposal, or roadmap before committing to it, and return one short, scannable verdict. Apply this whenever the user shares or points to a plan or proposal and asks whether it is sound, what is missing, what could go wrong, whether the approach is right, or whether to proceed — and on phrasings like “review this plan”, “poke holes in this”, “sanity-check this”, “what am I missing”, “is this a good plan”, or “should we do this”. Use it even when the user does not say the word “review”, as long as they are weighing a plan or about to act on one. It runs five reviewer lenses (right problem, tradeoff, blast radius, measurable success, the call) and consolidates them into a single verdict. It does not assign a numeric score or rewrite the plan.

# PlanProof

Read a plan and hand back the shortest review that lets the reader decide whether to proceed. (This is PlanProof’s **review pass** — it does not score or rewrite the plan; see Scope.) Lead with the one line that survives skimming, then give one sharp finding per lens — never a list of every possible risk. A clean plan should get a short “proceed”, not manufactured problems.

The plan may be pasted into the message or provided as a file or path. Read it fully before reviewing.

## The five lenses

Apply each to the plan and keep only the single most important thing it surfaces (or “fine” if nothing material).

- **Right problem** — Is this solving the real problem, or a symptom? What is the most fragile assumption the plan rests on?
- **Tradeoff** — Of scope, time, and quality, which corner is this plan quietly sacrificing? Name it; an unpriced tradeoff is the one that bites.
- **Blast radius** — How big are the changes, and how reversible? Rate the riskiest step on one scale: **Easy** (instantly undone), **Costly** (real work to unwind), or **One-way door** (effectively permanent). Flag any one-way door without a rollback.
- **Measurable success** — Is “done” or “worked” defined with a number, a window, and a baseline? If goals are vague (“improve”, “faster”), success cannot be told from noise.
- **The call** — Cutting through the length: what is the actual decision right now, in one line?

## Output format

Use this shape. Drop any line that does not apply; never pad.

**Verdict:** [one line: proceed / proceed with fixes / rework, plus the single reason why. Lead with the answer.]

**By lens:**

- **Right problem:** [one line, or “fine”]
- **Tradeoff:** [one line, or “none material”]
- **Blast radius:** [one line using Easy / Costly / One-way door, or “Easy — low risk”]
- **Measurable success:** [one line, or “defined”]
- **The call:** [one line]

**Top fix:** [the single highest-leverage change. Omit if the verdict is a clean proceed.]

**Do next:** [one concrete move that reduces the biggest unknown. If it is genuinely 2-3 discrete pre-work items, a tiny checklist is fine — hyphen-prefixed boxes (`- [ ]`), never bare `[ ]`. Otherwise one line.]

## Calibration

- **One finding per lens, not five.** Name the single thing that changes the call. If a lens finds nothing material, say “fine” in a few words and move on.
- **Stay quiet when the plan is sound.** A small, well-scoped, reversible plan gets “Proceed — low risk” with most lenses saying “fine”. Do not invent issues to look thorough; that is the failure mode, not thoroughness.
- **Challenge the frame, not just the steps.** The strongest finding is often “this may be the wrong problem”, not “step 3 is risky”.
- **Do not fabricate facts.** If the plan omits something the review needs, say it is missing — do not fill the gap with a guess.
- **Reversibility is one vocabulary.** Always Easy / Costly / One-way door, so a two-way door reads differently from a commitment that is expensive to unwind.

## Scope of this skill

This is PlanProof’s **review** pass only. It does not assign a numeric completeness score (PlanProof’s rubric — planned) and it does not rewrite or auto-improve the plan (PlanProof’s improvement loop — planned). It tells the reader where the plan stands; the reader decides.

In the full PlanProof product, the five lenses are sourced read-only from the Giant Brains Claude Skills toolkit. Here their questions are summarized inline so the review runs standalone.

## Examples

**Example A — a flawed but normal-looking plan**

Input (abridged): a loyalty-points launch plan — vague goals (“improve retention”), a holiday deadline, a one-time backfill across all customer records, and a launch-day checkout switch, two part-time engineers, three weeks.

Output:

> **Verdict:** Proceed with fixes — buildable by the holidays, but two steps are hard to undo with no rollback, and no goal is measurable, so you would not know if it worked.
> 
> **By lens:**
> 
> - **Right problem:** Flat repeat-rate is the symptom; points assume the cause is “too little reward to return”. If the driver is product or delivery, points will not move it.
> - **Tradeoff:** Date fixed, scope full — so quality (testing the migration and checkout) is the corner most likely to give quietly. Decide now what you cut if week 3 slips.
> - **Blast radius:** Two one-way doors. The all-customer backfill is Costly to reverse; the launch-day checkout switch is a One-way door during peak season. Neither has a rollback.
> - **Measurable success:** No goal has a number. “Improve retention” needs a metric, window, and baseline (e.g., 60-day repeat rate vs. last quarter).
> - **The call:** Do not pause — close the reversibility gaps and define one metric, then build.
> 
> **Top fix:** Make both risky steps reversible — backfill into a droppable, recomputable ledger (not customer records), and put the new checkout behind a flag you can switch off instantly.
> 
> **Do next:**
> 
> - [ ] Write one success line: metric + window + baseline.
> - [ ] Add a rollback for the backfill and the checkout switch.
> - [ ] Check that points address why repeat-rate is actually flat.

**Example B — a small, sound change (stay quiet)**

Input (abridged): add a “sort by price” option to a product list, behind a feature flag, with a click-through metric and a one-week A/B test.

Output:

> **Verdict:** Proceed — low risk, well scoped.
> 
> **By lens:**
> 
> - **Right problem:** fine.
> - **Tradeoff:** none material.
> - **Blast radius:** Easy — flag-gated, instantly reversible.
> - **Measurable success:** defined (click-through, one-week A/B).
> - **The call:** ship it.
> 
> **Do next:** Build behind the flag as planned.

## What success looks like

The reader can decide whether to proceed in about 20 seconds, sees the one finding that most changes the call, and a sound plan is not buried under invented concerns.

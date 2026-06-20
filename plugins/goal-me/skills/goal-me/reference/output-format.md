# goal-me output format

Write files under `.claude/goals/<slug>/` (`<slug>` is a kebab-case name for the goal, e.g. `auth-jwt-migration`, `competitor-research-2026`).

`/goal` does not auto-read these files. The **completion-condition checklist in `GOAL.md` is the source of truth**, and the kickoff condition points at it (see below). The files hold the detailed spec, path, and memory.

**Language**: write the *content* of these files in the user's language (e.g. Japanese for this user). The templates below are scaffolding — translate the labels/placeholders to match.

## Scale the file count to the size of the goal (KISS)
Vary which files you create based on goal size. Do not always create 3.
- **Small (finishes in one session / 2–3 steps / no carry-over across runs)**: `GOAL.md` only. Put steps as a bullet list inside GOAL.md and track progress there with `- [ ]`. Do not create STEPS/PROGRESS.
- **Large (multi-session / 4+ steps / state carried across runs)**: `GOAL.md` + `STEPS.md` + `PROGRESS.md`.
- When in doubt, start small.

Map the loop's 5 elements (Goal / Context / Action / Feedback / Stop condition) onto GOAL.md (plus STEPS/PROGRESS when large).

---

## GOAL.md (the closed-loop definition = entry point for /goal)

```markdown
# Goal: <one sentence describing the achieved state>

## Completion conditions (Stop condition)
<Write ONLY machine-checkable conditions. /goal loops until these are "actually true".>
- [ ] <observable condition 1>  Verify: `<command or check procedure>`
- [ ] <observable condition 2>  Verify: `<command or check procedure>`

Example (dev): `pytest tests/auth/ all pass, ruff check . reports 0 errors`
Example (research): `all 5 key claims cite >=2 primary sources; conflicting info presented both ways`

## Scope
### In
- <included>
### Out (explicitly excluded / prevents cost blowup)
- <excluded. Skip this and the loop opens up and cost balloons.>

## Context
<Existing info the loop should reference. For new work, the premises; for existing, paths to the relevant files/docs.>
- References: <path or URL or premise>
- Constraints / rules: <conventions to follow, things forbidden>

## Cost guardrails
- Step granularity: <unit that finishes in one session / one verify cycle>
- Size ceiling: <file count / iteration count / token budget, etc.>
- Escalation: <condition that returns control to a human>

<!-- For a small goal (single GOAL.md), also keep the following inside GOAL.md.
     For a large goal, split these into STEPS.md / PROGRESS.md and omit them here. -->
## Steps (small goals only, in dependency order)
1. <name> — Verify: `<gate>`
2. <name> — Verify: `<gate>`

## Progress (small goals only)
- [ ] 1. <name>
- [ ] 2. <name>
```

---

## STEPS.md (the defined path = an eval gate per step)

```markdown
# Execution steps

Each step has "Goal → Action → Verify (eval gate) → Done condition".
Order by dependency. One step = the smallest unit where verification runs once.

## Step 1: <name>
- Depends on: none
- Goal: <achieved state after this step (one sentence)>
- Action: <what the agent actually does>
- Verify (eval gate): `<command / test / grep / numeric threshold / checklist>`
- Done condition: <what verification result lets you move on (no fuzzy words)>

## Step 2: <name>
- Depends on: Step 1
- ...
```

---

## PROGRESS.md (memory = carries state across runs)

```markdown
# Progress

The loop forgets between runs. The repo (this file) does not.
/goal and /loop read this to carry forward "how far we got / what passed / what's still open".

## State
- [ ] Step 1: <name> — pending
- [ ] Step 2: <name> — pending

## Log
(appended by runs. e.g. 2026-06-20 Step1 verify pass / Step2 lint fail -> open)

## Open / handover
-
```

---

## Good vs. bad goals (by domain)

Verification must be objectively judgeable by someone else (a different model). Maker ≠ Checker.

| Domain | ❌ Bad (fuzzy / opens the loop) | ✅ Good (machine-checkable) |
|---|---|---|
| Dev | clean up the code / improve auth | `pytest tests/auth all pass AND ruff/mypy 0 errors AND login API returns 401->200` |
| Bug fix | fix the bug | `add a repro test from the issue's steps where the exception no longer fires, and it passes` |
| Research | look into competitors | `a 5-competitor x (price/key features/differentiator) table fully filled, each cell with >=1 primary-source URL; stop when no new source appears for 2 consecutive passes` |
| Writing | write a good article | `target reader = X, outline covers N required points, each claim has evidence, all proofreading-checklist items cleared` |
| Investigation | understand the whole thing | `output a list of the target dir's main modules and a dependency diagram, identify the entry point with the evidence file path` |

## Kickoff command (/goal takes the condition as an argument)
`/goal` does not auto-read `.claude/goals/`. You pass the stop condition via `/goal <condition>`, and it loops until that is true.
Important: **the condition is judged true/false by a different model (judge) from observable state** (Maker ≠ Checker). The condition itself must be a statement whose truth is determinable.
- ❌ A bare path (e.g. `/goal .claude/goals/x/`) is ambiguous to the judge — what counts as "true and stop" is unknown.
- ✅ Put "which file to read" + "machine-checkable stop condition" + "how to proceed" into one sentence.

After writing files, always present a copy-pasteable one-liner (adjust wording to size):

```
/goal Read .claude/goals/<slug>/GOAL.md and implement until its "Completion conditions" checklist is all true. <If large: follow STEPS.md in order and update PROGRESS.md as each step completes.> If <escalation condition>, stop and ask.
```

GOAL.md's completion-condition checklist is the source of truth; the condition is just the entry pointing to it. Keep completion conditions self-contained in GOAL.md so that even if shortened, completion is still judgeable from GOAL.md alone.

## Fuzzy-word blacklist (when detected, always make concrete)
clean / appropriate / nice / improve / optimize / properly / solid / as needed / high confidence / somewhat / sufficiently / comprehensively
→ Always restate these as "what, measured how, counts as done". If you can't restate it, it isn't a stop condition.
→ To express "we've searched enough" (research, etc.), avoid subjective words; use an observable convergence condition like `no new source/finding for N consecutive passes`.

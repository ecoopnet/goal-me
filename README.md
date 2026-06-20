# goal-me

*[日本語版 README はこちら](README.ja.md)*

A Claude Code skill that turns a vague "I want to build / research X" into a **machine-verifiable closed loop**, so the built-in `/goal` command can run it **without drifting or burning excess cost**.

It is inspired by the simplicity of `grill-me` and by *loop engineering*: an agent runs `Goal → Action → Check → Fix → Repeat until done`, and `/goal` keeps looping **until a condition you wrote is actually true**. If that condition is fuzzy, the loop wanders (an "open loop") and burns tokens. `goal-me` interviews you first to design a **narrow, closed** path with a precise, objectively checkable stop condition.

---

## Why use it

`/goal` is powerful but only as good as the condition you hand it. Give it "make the code clean" and it will drift or run forever. Give it "`pytest tests/auth` all pass and `ruff` reports 0 errors" and it knows exactly when it is done.

`goal-me` is the front end that produces that kind of condition for you — by asking the right questions, banning fuzzy words, and forcing every "done" into something a *different* model can verify (Maker ≠ Checker).

It works for **development, research, writing, and investigation**, on **new or existing projects**.

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with plugin support (`/plugin`).
- The built-in `/goal` command (used to run the loop after `goal-me` designs it). `/goal-me` itself only designs and writes files; running the loop is a separate, optional step.

---

## Install

```
/plugin marketplace add ecoopnet/goal-me
/plugin install goal-me
```

Verify it is installed:

```
/plugin            # goal-me should appear in the installed list
/help              # /goal-me should be listed as a command
```

### Manual install (alternative)

If you prefer not to use the marketplace, copy the skill folder into your skills directory:

```
git clone https://github.com/ecoopnet/goal-me
cp -r goal-me/plugins/goal-me/skills/goal-me ~/.claude/skills/
```

### Update / uninstall

```
/plugin marketplace update ecoopnet/goal-me   # pull the latest
/plugin uninstall goal-me                      # remove it
```

---

## Quickstart

1. Start a design session with whatever you have, however vague:

   ```
   /goal-me I want a small web tool that calculates portfolio risk, mobile-friendly
   ```

2. **Answer the interview.** It asks one question at a time, each with a recommended answer. It will also read your repo/files to answer what it can without bothering you. The questions converge on:
   - what "done" means, as an **observable fact**;
   - **how it is verified mechanically** (test / lint / typecheck / grep / numeric threshold / source count / checklist);
   - what is **out of scope** (this is what keeps cost down);
   - cost guardrails and an escalation condition.

3. **It writes files** under `.claude/goals/<slug>/`, scaled to the size of the goal:
   - small goal → a single `GOAL.md`;
   - large goal → `GOAL.md` + `STEPS.md` + `PROGRESS.md`.

4. **It gives you a kickoff command** to paste — something like:

   ```
   /goal Read .claude/goals/<slug>/GOAL.md and implement until its "Completion conditions" are all true. If <escalation condition>, stop and ask.
   ```

5. **Run it.** `/goal` loops until the condition holds, then stops on its own. For a scheduled re-run, use `/loop`.

---

## What gets written

```
.claude/goals/<slug>/
├── GOAL.md       # the source of truth: completion conditions, scope, context, cost guardrails
├── STEPS.md      # (large goals) the path: each step with its own eval gate
└── PROGRESS.md   # (large goals) memory carried across runs
```

`GOAL.md`'s **completion-condition checklist is the source of truth**. `/goal` does not read these files automatically — the kickoff command points it at them. Keep the conditions self-contained in `GOAL.md` so completion stays judgeable even if you shorten the command.

---

## Design principles

- **Closed loop by default.** A human designs a narrow path first; open-ended exploration balloons cost.
- **Machine-checkable stop conditions only.** Fuzzy words ("clean", "improve", "high confidence") are banned and forced concrete.
- **Maker ≠ Checker.** The stop condition is judged by a different model from observable state, so it must be objectively determinable — never the executor self-grading.
- **Scale the artifacts to the goal.** Don't generate three files for a one-session task.

---

## Layout

```
goal-me/
├── .claude-plugin/marketplace.json     # marketplace manifest
└── plugins/goal-me/
    ├── .claude-plugin/plugin.json      # plugin manifest
    └── skills/goal-me/
        ├── SKILL.md                    # behavior (English; the interview and generated files follow the user's language)
        └── reference/output-format.md  # file templates + good/bad goal examples
```

## License

MIT — see [LICENSE](LICENSE). Copyright is retained by the author; the license only grants usage.

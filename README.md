# goal-me

A Claude Code skill that turns a vague "I want to build/research X" into a **machine-verifiable closed loop** so the built-in `/goal` can run it **without drifting or burning excess cost**.

Inspired by the simplicity of `grill-me` and by *loop engineering* (`Goal → Action → Check → Fix → Repeat until done`): a human designs a narrow, closed path first, and the loop runs until a condition that is **actually true**.

## What it does

`/goal-me <what you want>` interviews you one question at a time (with a recommended answer for each), then:

1. **Frames** the domain (dev / research / writing / investigation) and new-vs-existing.
2. **Interviews** to extract a *precise, machine-checkable stop condition* — fuzzy words ("clean", "improve", "high confidence") are banned and forced concrete.
3. **Designs a closed loop**: bounded steps in dependency order, each with its own eval gate.
4. **Writes files** under `.claude/goals/<slug>/` (scaled to size: one `GOAL.md` for small goals, `GOAL.md`+`STEPS.md`+`PROGRESS.md` for large).
5. **Produces a kickoff command** — a ready-to-paste `/goal <condition>` one-liner. (`/goal` takes the stop condition as an argument; it does not auto-read files.)

The core value: the stop condition is judged by a *different* model from observable state (Maker ≠ Checker), so it must be objectively determinable. That is what keeps `/goal` from wandering.

## Install (plugin marketplace)

```
/plugin marketplace add ecoopnet/goal-me
/plugin install goal-me
```

Then run:

```
/goal-me I want to build a small web tool that ...
```

## Manual install (alternative)

Copy the skill into your skills directory:

```
git clone https://github.com/ecoopnet/goal-me
cp -r goal-me/plugins/goal-me/skills/goal-me ~/.claude/skills/
```

## Layout

```
goal-me/
├── .claude-plugin/marketplace.json     # marketplace manifest
└── plugins/goal-me/
    ├── .claude-plugin/plugin.json      # plugin manifest
    └── skills/goal-me/
        ├── SKILL.md                    # behavior (English; interview/output follow the user's language)
        └── reference/output-format.md  # file templates + good/bad goal examples
```

## License

MIT — see [LICENSE](LICENSE).

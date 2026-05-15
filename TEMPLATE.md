# Adding overlay support to a baseline skill

Any skill in this plugin can be made **layerable** by following the conventions below. The mechanism is intentionally tiny: one section in the baseline, one fixed slot path, no hooks or agents.

## The contract

A baseline skill is layerable when it:

1. Lives in this plugin at `plugins/skill-layers/skills/<name>/SKILL.md` and is fully usable on its own as shipped.
2. Documents a fixed overlay slot at `.claude/skills/<name>-overlay/SKILL.md` in the consumer repo.
3. Reads that overlay file (if present) before producing output, and treats its body as additional, equally-binding rules.
4. Never lets the overlay remove, soften, or override a baseline rule. Conflicts are surfaced, not silently accepted.

## Copy-paste boilerplate

Paste this section at (or near) the top of any baseline SKILL.md you want to make layerable. Replace `<name>` with the baseline's directory name.

```markdown
## Load the project overlay (if present)

Before producing output, check whether `.claude/skills/<name>-overlay/SKILL.md` exists in the working repository. If it does, read it and treat the rules in its body as additional, equally-binding constraints alongside everything below. Ignore the file's YAML frontmatter when applying its rules.

The overlay may:

- add new rules,
- narrow scope,
- specify project conventions, naming, or stack-specific guidance.

The overlay may NOT remove, soften, or override any rule in this baseline. If the overlay appears to contradict a baseline rule, the baseline rule wins and the conflict must be surfaced in the output.
```

And add this as a final section, so the model verifies it actually applied the overlay:

```markdown
## Self-check before finalising

Before returning, confirm:

- Every baseline rule was applied.
- If an overlay was present, every overlay rule was applied too.
- If the overlay caused you to skip any baseline rule, that omission is surfaced as a conflict — not silently accepted.
```

## Naming rules

- **Baseline skill name:** plain noun or verb-phrase (`code-review`, `commit-message`, `research`). No suffix.
- **Overlay slot:** `<baseline>-overlay`. Always `-overlay`, never `-local`, `-project`, or `-base`.
- **Invocation:**
  - Baseline runs as `/skill-layers:<baseline>` (plugin-namespaced).
  - Overlay runs as `/<baseline>-overlay` (consumer project skill).
  - Both are independently invocable. The overlay is also auto-loaded as data when the baseline runs.

## Writing a good overlay

When you write an overlay in a consumer repo:

- The body must be **self-contained**. A developer reading just the overlay should understand each rule without the baseline open.
- Phrase rules as additions, not as edits to baseline sections. `"In this repo, also …"` is better than `"Replace section 4 with …"`.
- Keep it short. The overlay is a project layer, not a fork of the baseline.

## Layers stack, they don't shadow

The mental model: the baseline is the floor everyone stands on; the overlay is the project-specific layer on top. The floor never moves. If a project wants to weaken a baseline rule, that's a signal the rule belongs in a different skill — not an excuse to override.

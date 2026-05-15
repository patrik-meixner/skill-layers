# Adding overlay support to a baseline skill

Any skill in this plugin can be made **layerable** by following the conventions below. The mechanism is intentionally tiny and uses Claude Code's built-in dynamic context injection so the overlay is preloaded by the harness before the model sees the skill — there is no soft "first action" rule to be skipped.

## The contract

A baseline skill is layerable when it:

1. Lives in this plugin at `plugins/skill-layers/skills/<name>/SKILL.md` and is fully usable on its own as shipped.
2. Documents a fixed overlay slot at `.claude/skills/<name>-overlay/SKILL.md` in the consumer repo.
3. **Preloads** that overlay file (if present) into its own prompt using a `` !`<command>` `` injection — not a runtime Read tool call. Absence yields the sentinel `NO_OVERLAY`.
4. Treats the loaded overlay rules as additional, equally-binding rules. Never lets the overlay remove, soften, or override a baseline rule. Conflicts are surfaced, not silently accepted.

## Copy-paste boilerplate

Paste this into any baseline SKILL.md you want to make layerable. Replace `<name>` (twice) with the baseline's directory name.

### Frontmatter

Make sure the baseline declares the allowed tools for the injection so it runs without prompting:

```yaml
---
description: ...
allowed-tools: Bash(cat *) Bash(git rev-parse *)
---
```

### Overlay block

Place this section near the **top** of the body, before the baseline rules:

````markdown
## Project overlay (preloaded)

The block between `--- BEGIN OVERLAY ---` and `--- END OVERLAY ---` was preloaded from `.claude/skills/<name>-overlay/SKILL.md` in the working repository.

- If the block reads exactly `NO_OVERLAY`, no overlay is configured. Proceed with the baseline rules below only. Do **not** warn, do **not** suggest creating one, and do **not** include any overlay-related sections in the output.
- Otherwise, treat every rule in the block as an **additional, equally-binding constraint** alongside the baseline rules below. Ignore any YAML frontmatter that appears inside the block.

The overlay may add new rules, narrow scope, or specify project conventions. The overlay may **not** remove, soften, or override any rule in this baseline. If the overlay appears to contradict a baseline rule, the baseline rule wins and the conflict must be surfaced in the output under a "Conflicts with overlay" section.

--- BEGIN OVERLAY ---
!`cat "$(git rev-parse --show-toplevel 2>/dev/null)/.claude/skills/<name>-overlay/SKILL.md" 2>/dev/null || echo "NO_OVERLAY"`
--- END OVERLAY ---
````

### Self-check footer

Place this as the final section of the body, so the model verifies it applied the overlay:

```markdown
## Self-check before finalising

Before returning, confirm:

- Every baseline rule was applied.
- If an overlay was loaded (i.e. the block above was not `NO_OVERLAY`), every overlay rule was applied too.
- If the overlay caused you to skip any baseline rule, that omission is surfaced as a conflict — not silently accepted.
```

## Why preloading instead of "first action"

`` !`<command>` `` runs **before** the model sees the skill. The harness performs the read; the model has the overlay (or `NO_OVERLAY`) already inlined in its prompt. There is no ordering decision to get wrong and no tool call for the model to forget. The `git rev-parse` prefix is so the injection finds the overlay even when Claude Code was started from a subdirectory.

## Naming rules

- **Baseline skill name:** plain noun or verb-phrase (`code-review`, `commit-message`, `research`). No suffix.
- **Overlay slot:** `<baseline>-overlay`. Always `-overlay`, never `-local`, `-project`, or `-base`.
- **Invocation:**
  - Baseline runs as `/skill-layers:<baseline>` (plugin-namespaced) — overlay preloaded automatically.
  - Overlay runs as `/<baseline>-overlay` (consumer project skill) — standalone, project rules only.

## Writing a good overlay

When you write an overlay in a consumer repo:

- The body must be **self-contained**. A developer reading just the overlay should understand each rule without the baseline open.
- Phrase rules as additions, not as edits to baseline sections. `"In this repo, also …"` is better than `"Replace section 4 with …"`.
- Keep it short. The overlay is a project layer, not a fork of the baseline.

## Layers stack, they don't shadow

The mental model: the baseline is the floor everyone stands on; the overlay is the project-specific layer on top. The floor never moves. If a project wants to weaken a baseline rule, that's a signal the rule belongs in a different skill — not an excuse to override.

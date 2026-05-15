# skill-layers

A Claude Code plugin that ships **baseline skills** with a standardised **overlay** mechanism. The baseline gives you the general practice; your repo can add a project-specific overlay that the baseline auto-loads and applies on top.

**Layers stack, they don't shadow.** An overlay can add and narrow; it can never remove or soften a baseline rule.

## How it works

Each baseline skill in this plugin documents one fixed overlay slot:

```
.claude/skills/<baseline>-overlay/SKILL.md
```

When the baseline is invoked, Claude Code **preloads** the overlay file into the baseline's prompt using a dynamic-context-injection command — before the model runs. If the file is missing, the sentinel `NO_OVERLAY` is loaded instead. The baseline then applies any loaded overlay rules as **additional, equally-binding** constraints. Conflicts (overlay tries to override a baseline rule) are surfaced in the output, not silently accepted.

Preloading (rather than asking the model to "Read the overlay first") means the overlay arrives in context deterministically. There is no ordering for the model to get wrong.

The overlay is also a normal Claude Code skill, so it is **independently invocable**:

| Invocation                  | What runs                                |
| :-------------------------- | :--------------------------------------- |
| `/skill-layers:code-review` | Baseline + overlay (if present)          |
| `/code-review-overlay`      | Overlay only (quick project-rules check) |

No hooks. No agents. No state files. Just one section in each baseline and one fixed file path.

## Install

```text
/plugin marketplace add patrik-meixner/skill-layers
/plugin install skill-layers@skill-layers
```

## What ships

Currently one baseline skill, intentionally:

- **`code-review`** — context-detected static analysis, stack-agnostic security checklist, code-level checks, clean-code audit. Auto-loads `.claude/skills/code-review-overlay/SKILL.md` if present.

More baselines will be added once the layering convention has been used in anger on a real project.

## Add an overlay to your repo

1. Create the slot:

   ```bash
   mkdir -p .claude/skills/code-review-overlay
   ```

2. Drop a `SKILL.md` there. Start from [`examples/code-review-overlay/SKILL.md`](examples/code-review-overlay/SKILL.md) and edit.
3. That's it. Next time you run `/skill-layers:code-review`, the overlay is applied automatically. You can also run `/code-review-overlay` standalone for a quick project-conventions check.

## Author a new layerable skill

See [`TEMPLATE.md`](TEMPLATE.md) for the contract, the copy-paste boilerplate, and naming rules.

## License

MIT.

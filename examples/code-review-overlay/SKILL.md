---
description: Project-specific code-review rules for THIS repo. Loaded automatically by skill-layers:code-review as an overlay, and also invocable standalone for a quick project-conventions check.
---

# Code Review — Project Overlay (example)

These rules are project-specific and apply **in addition to** the rules in `skill-layers:code-review`. They never replace or weaken the baseline.

When invoked standalone, only the rules below run — useful for a quick conventions check without the full security/clean-code pass.

## Naming

- Repository names and module names use `kebab-case`.
- React component files use `PascalCase` and one component per file.
- Test files live next to the unit under test as `<name>.test.ts`.

## Layering

- Code in `src/ui/` may import from `src/lib/`, never the other way around.
- Database access only goes through `src/db/`. No raw SQL elsewhere.

## Testing

- New public functions ship with at least one test covering the golden path and one edge case.
- Snapshot tests are not accepted as the only test for a component.

## Commits

- Commit subjects follow Conventional Commits (`feat:`, `fix:`, `refactor:`, …).
- One logical change per commit; no "wip", "fix typo in last commit" rolled together.

---

This file is an **example overlay**. To enable it in your project, copy it (or write your own) to:

```
.claude/skills/code-review-overlay/SKILL.md
```

at the root of your repository.

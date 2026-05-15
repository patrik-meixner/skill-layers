---
description: Baseline code review for the current changes. Runs context-detected static analysis, a stack-agnostic security checklist, and a clean-code audit. Automatically merges project-specific rules from .claude/skills/code-review-overlay/SKILL.md when present. Use when the user asks for a code review, wants to vet uncommitted changes, or is preparing to open a PR.
---

# Code Review (baseline)

Review the pending changes against the rules below. Project conventions can extend this baseline via an **overlay** (see the final section); do not skip the overlay step if it is present.

## 1. Load the project overlay (if present)

Before reviewing, check whether `.claude/skills/code-review-overlay/SKILL.md` exists in the working repository.

**If it does not exist:** proceed with the baseline rules below only. Do not warn, do not suggest creating one, do not include any overlay-related sections in the output. The baseline is fully usable on its own.

**If it does exist:** read it and treat the rules in its body as additional, equally-binding constraints alongside everything below. Ignore the file's YAML frontmatter when applying its rules.

The overlay may:

- add new rules,
- narrow scope (e.g. "in this repo, also check X"),
- specify project conventions, naming, or stack-specific guidance.

The overlay may NOT remove, soften, or override any rule in this baseline. If the overlay appears to contradict a baseline rule, the baseline rule wins and the conflict must be surfaced in the output under a "Conflicts with overlay" section.

## 2. Discover what already exists

Do not re-implement in this review what a project-specific tool or skill already owns. Before running checks:

- Read `CLAUDE.md` (and any nested `CLAUDE.md`) for prescribed commands.
- Look at the project's manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, etc.) for `check`, `verify`, `lint`, `test`, `ci` scripts.
- Note any project skills under `.claude/skills/` whose descriptions overlap with this review.

Prefer running the project's own commands over re-deriving rules.

## 3. Static analysis (context-detected)

Run only tooling the project already configures. In priority order:

1. Aggregate commands authored by the project (`check`, `verify`, `ci`).
2. Per-tool scripts in the package manifest.
3. Configured tools whose configs exist in the repo.

If no static-analysis tooling is configured, record this as a finding ("project has no configured static analysis") and continue with the human-verified checks below. Do **not** install or introduce new tooling as part of the review.

## 4. Security checklist (stack-agnostic)

- **Input handling:** validation at boundaries, no dynamic eval, parameterised queries, path normalisation, safe subprocess invocation, robust parsers.
- **Output handling:** context-aware escaping, sanitised errors and logs.
- **Auth & authz:** explicit checks at every entrypoint, per-request verification, centralised policy.
- **Secrets & deps:** no hardcoded credentials, no committed `.env`, dependency vulnerabilities triaged, licences compatible.
- **Crypto & tokens:** platform libraries only, constant-time comparisons, sensible expiry.
- **Business logic:** race conditions, overflow/underflow, pagination correctness, idempotency where required.

## 5. Code-level checks (linters often miss these)

- Cyclomatic complexity ≤ 10 branches per function.
- Nesting depth ≤ 3.
- Layering and cohesion: no upward dependencies, no god-modules.
- Dead code removed.
- Error paths take purposeful action (not silently swallowed, not blanket-rethrown).
- No obvious N+1 queries.
- No unbounded memory growth (lists/maps that only grow).

## 6. Clean-code audit

- Names reveal intent: a stranger understands each identifier without the surrounding context.
- Single responsibility per function; if you cannot name it without "and", split it.
- Comments explain **why**, not **what**. Drop comments that only restate the code.
- No scope creep: the diff does only what the task says.
- No TODOs, debug prints, debuggers, or commented-out code left behind.
- Convention alignment: the diff matches the style of files around it.

## 7. Output

Produce a single review report with these sections:

1. **Summary** — one paragraph, what changed and the overall verdict.
2. **Blocking findings** — issues that must be fixed before merge, each with file + line.
3. **Non-blocking suggestions** — nits and polish.
4. **Overlay findings** (only if an overlay was loaded) — findings produced by the project-specific rules, labelled so the reader can tell them apart.
5. **Conflicts with overlay** (only if the overlay contradicted a baseline rule) — describe the conflict and note that the baseline rule was applied.

## 8. Self-check before finalising

Before returning the report, confirm:

- Every section above was applied.
- If an overlay was present, every overlay rule was applied too.
- If the overlay caused you to skip any baseline rule, that omission is surfaced as a conflict — not silently accepted.

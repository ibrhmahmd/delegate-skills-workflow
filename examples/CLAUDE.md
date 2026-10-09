# Delegate-Skills Workflow — Mandatory, Applies to Every Project

This project uses **delegate-skills** for a Claude (planning/review) + OpenCode
(implementation) workflow. Skills and fleet config are global
(`~/.agents/skills/`, `~/.config/delegate-fleet.json`), so this workflow
applies in every project, not just where it was first set up.

## Fleet

5 lanes, all bound to OpenCode (`opencode/nemotron-3-ultra-free`, set in `~/.config/delegate-fleet.json`):
`feature`, `refactor`, `ui`, `docs`, `performance`.

No Cursor, no Codex — user works with Claude (orchestrator/planner/reviewer)
and OpenCode (implementer) only.

## The 7-Phase Workflow Is Mandatory

Every non-trivial task follows all 7 phases. No skipping phases, no manual
implementation outside delegation, no merging without review.

1. **Issue Creation** — every task needs a tracking issue first (GitHub
   Issues, Linear, or a local `ISSUES.md` in the repo if nothing else is
   set up). Tag it: feature / bug / refactor / docs / performance.

2. **Analysis & Planning** — use `$domain-modeling`, `$codebase-design`,
   `$wayfinder`, `$to-spec`, `$decision-mapping` as fits. Use `$wait-what`
   style clarification whenever the ask is ambiguous — confirm the actual
   symptom/requirement before proceeding, don't guess.

3. **Implementation Planning** — use `$request-refactor-plan` /
   `$improve-codebase-architecture` / `$tdd`. **Discuss the plan with the
   user before delegating** — implementation plans are not unilateral,
   especially for anything architectural.

4. **Delegate to OpenCode** — via `$feature-delegate`, `$refactor-delegate`,
   `$ui-delegate`, `$docs-delegate`, or `$performance-delegate`. Give
   OpenCode a precise, scoped brief referencing exact files/functions found
   during diagnosis — not a vague restatement of the issue. OpenCode should
   leave changes **uncommitted** for review unless told otherwise.
   - When invoking `opencode run` from the shell, **write the prompt to a
     file and pass it with `--file`** rather than inlining it in a
     double-quoted string — backticks, `$(...)`, and parens in the prompt
     text will get interpreted by the shell and silently mangle the prompt.

5. **Code Review** — this is the real quality gate, not a formality:
   - Use `$code-review` (mattpocock primary, anthropics as a second opinion
     when useful) and `$grill-me` for complex/suspicious sections.
   - **Actually run the tests yourself** — don't just trust the
     implementer's self-report that "all tests pass." Re-run independently.
   - If tests need a live DB and the configured one is unreachable, spin up
     a **local Postgres** (schema.sql + migrations) rather than skipping
     verification or pointing tests at production. Test files that INSERT/
     COMMIT rows must never run against production — only read-only checks
     are acceptable there, and only with explicit user approval.
   - When re-running failing tests to check if a delegated change caused a
     regression, verify with `git stash` against the original code — if the
     same tests fail on unmodified code, it's pre-existing, not a
     regression.
   - Report every finding back with file:line precision. Real bugs get
     re-delegated with a precise, itemized fix brief (same file-based
     prompting discipline as phase 4).

6. **Merge & Close** — user approves and lands the commit. Issue closes via
   commit message (`closes #N` / `fixes #N`). Never assume approval — ask.

7. **Tracking & Follow-up** — use `$to-tickets` to capture follow-up work
   discovered during implementation. Use `$adr-author` / `$handoff` /
   `$to-prd` as fits.

## Operational vs. Code Work

Some fixes have a piece only the user can do (rotating credentials, cloud
dashboard cron setup, DNS, etc.). Don't try to delegate those to OpenCode —
use `$wizard` to walk the user through them, and keep the code-only portion
scoped separately in the delegation brief.

## Diagrams

Only generate architecture diagrams on demand — ask first, don't
auto-generate them as part of every planning phase.

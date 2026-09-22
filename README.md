# Delegate-Skills Workflow

A structured, mandatory 7-phase workflow for software teams using Claude (planning/review) + OpenCode (implementation). Ensures every task follows a consistent, quality-gated process from issue creation through deployment.

**Status:** Production-tested (used across multiple projects)  
**Last Updated:** 2026-09-22

## Quick Start

1. **Install skills** (one-time):
   ```bash
   npx skills add amElnagdy/delegate-skills
   ```

2. **Copy fleet config** to `~/.config/delegate-fleet.json`:
   ```bash
   cp examples/delegate-fleet.json ~/.config/
   ```

3. **Copy global instructions** to `~/.claude/CLAUDE.md`:
   ```bash
   cp examples/CLAUDE.md ~/.claude/
   ```

4. **Read the workflow** — understand the [7-phase process](WORKFLOW.md)

5. **Start your first issue** — follow [Phase 1: Issue Creation](WORKFLOW.md#phase-1--issue-creation)

## The 7 Phases (In 30 Seconds)

| Phase | Owner | Outcome |
|-------|-------|---------|
| **1. Issue Creation** | You | Tracking issue tagged (feature/bug/refactor/docs/performance) |
| **2. Analysis & Planning** | Claude | Diagnosis of the actual problem (not assumptions) |
| **3. Implementation Planning** | Claude + You | Detailed plan discussed and approved before coding |
| **4. Delegate to OpenCode** | OpenCode | Code changes (uncommitted, ready for review) |
| **5. Code Review** | Claude (You verify) | Real quality gate — tests run, regressions checked |
| **6. Merge & Close** | You | Approval given, code lands, issue closes |
| **7. Tracking & Follow-up** | Claude | Capture operational/deferred work via `$to-tickets` |

**Key rule:** No skipping phases. No manual implementation outside delegation. No merging without review.

## Why This Workflow?

- **Consistent** — every team member follows the same process
- **Auditable** — every decision is tracked in issues + git commits
- **Quality-gated** — Phase 5 is a real barrier, not a formality
- **Scalable** — works for 1 person or a team of 10

## Project Structure

- **[WORKFLOW.md](WORKFLOW.md)** — Full 7-phase breakdown with examples
- **[SETUP.md](SETUP.md)** — Installation and configuration guide
- **[CASE-STUDY.md](CASE-STUDY.md)** — Real example: the techno-terminal daily report bug
- **[FAQ.md](FAQ.md)** — Common questions and answers
- **[templates/](templates/)** — Copy these into your own repos
- **[examples/](examples/)** — Reference files from production projects

## Skills Required

This workflow uses these Claude Code skills (installed globally):

- `delegate-setup` — initialize a new project
- `opencode-delegate` — hand off work to OpenCode
- `code-review` — review implementation
- `domain-modeling`, `codebase-design` — planning phases
- `to-tickets` — capture follow-up work
- And 10+ others for specific tasks

See [SETUP.md](SETUP.md#installed-skills) for the full list.

## Fleet Configuration

The workflow is bound to a **fleet** — a set of 5 lanes (feature, refactor, ui, docs, performance), each assigned to an implementer. Default: all bound to OpenCode with `grok-4.6`.

You can customize this in `~/.config/delegate-fleet.json`. See [SETUP.md](SETUP.md#fleet-configuration) for details.

## Getting Help

- **Questions about the workflow?** → Read [FAQ.md](FAQ.md)
- **Stuck on Phase 3?** → See [CASE-STUDY.md](CASE-STUDY.md#phase-3--implementation-planning)
- **Don't know which skill to use?** → Check [WORKFLOW.md](WORKFLOW.md#skills-by-phase)
- **Setting up for the first time?** → Follow [SETUP.md](SETUP.md)

## Contributing

Found an issue or want to improve the docs? Open an issue or PR on this repo.

## License

MIT — use this workflow freely in your projects.

---

**Next:** Start with [SETUP.md](SETUP.md) or jump to [WORKFLOW.md](WORKFLOW.md) to understand the phases.

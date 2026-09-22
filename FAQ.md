# Frequently Asked Questions

---

## Workflow

### Q: Do I really have to follow all 7 phases every time?

**A:** Yes. The workflow is mandatory, not optional. Even for small bug fixes.

Why? Because process discipline prevents mistakes that are expensive to fix later. A 20-minute Phase 2 diagnosis can prevent a 2-hour rabbit hole. A 30-minute Phase 5 review can catch bugs that would break production.

**Exception:** If the task is truly trivial (typo fix, update a comment, change a log level), you can land it without going through all phases. But use judgment — most things that *feel* trivial have hidden complexity.

### Q: Can I skip Phase 3 if the plan is obvious?

**A:** No. Even "obvious" fixes need a plan discussion. Reasons:

1. **You might be wrong** — what looks obvious often has surprises
2. **Audit trail** — you want to know why this approach was chosen
3. **Alignment** — the user approves before implementation starts

### Q: What if Phase 4 (implementation) produces code I don't like?

**A:** If the code is correct, passes tests, and follows the project's conventions, land it.

Code style is not a blocker. Correctness, tests, and maintainability are.

### Q: Can I do Phase 5 review quickly?

**A:** Not if "quickly" means skipping tests. Actually run the tests yourself. This is not negotiable.

Phase 5 is the quality gate. That's the whole point.

---

## Skills

### Q: Which skill should I use for X?

**A:** See [WORKFLOW.md#skills-by-phase](WORKFLOW.md#skills-by-phase) for a matrix.

If you're still unsure, ask Claude: "Which skill should I use for..."

### Q: How do I create my own skill?

**A:** See the [Claude Code documentation](https://claude.com/code) for the skill format.

Example skills are in this repo's `examples/` directory.

### Q: What if a skill doesn't exist yet?

**A:** Create an issue in this repo, or email the maintainer.

For now, the default skills installed via `npx skills add amElnagdy/delegate-skills` cover most cases.

---

## Implementation

### Q: What if OpenCode's code is wrong?

**A:** Don't land it. Use Phase 5 to flag the bugs, then re-delegate with a precise fix brief.

This happens sometimes. The re-delegation cycle is normal and expected.

### Q: Can I commit OpenCode's changes directly after Phase 4?

**A:** No. Phase 5 review is required before any merge.

The code stays uncommitted until review is done.

### Q: What if the tests take too long to run?

**A:** You still have to run them. No exceptions.

If tests are slow (>10 minutes), that's a separate technical debt issue. File it and move on. But you cannot skip test running.

### Q: How do I handle database-dependent tests?

**A:** If the configured test database is unreachable:

1. Don't skip tests (Phase 5 requirement)
2. Don't point them at production (dangerous)
3. Spin up a local Postgres with schema + migrations
4. Run tests locally

See [SETUP.md#troubleshooting](SETUP.md#troubleshooting) for details.

---

## Git & Commits

### Q: What if the commit gets rejected by a pre-commit hook?

**A:** Investigate the error. Don't skip the hook (`--no-verify`).

Fix the underlying issue (linting, formatting, type checking), then commit again.

### Q: Should I amend the previous commit or create a new one?

**A:** Create a new commit (don't amend).

If you're fixing bugs found in Phase 5, that's a new implementation round with a new commit. Don't rewrite history.

### Q: What commit message format should I use?

**A:** See [WORKFLOW.md#phase-6--merge--close](WORKFLOW.md#phase-6--merge--close).

Summary:
- First line: what changed (imperative: "fix", "add", "refactor")
- Body: 1-2 sentences explaining why
- Footer: `closes #N` (link to issue), `Co-Authored-By: Claude <noreply@anthropic.com>`

---

## Operations

### Q: What's the difference between this workflow and just writing code?

**A:** This workflow adds process discipline at these key points:

1. **Phase 2:** Diagnosis before implementation (avoid wrong fixes)
2. **Phase 3:** Plan approval before coding (alignment)
3. **Phase 5:** Real review with independent test runs (quality gate)
4. **Phase 7:** Tracking follow-up work (nothing gets forgotten)

It's overhead. But it catches mistakes that would be much more expensive to fix later.

### Q: Who owns each phase?

**A:** See [WORKFLOW.md](WORKFLOW.md#the-7-phases-in-30-seconds).

Summary:
- **You:** Phase 1 (issue), Phase 3 (discuss plan), Phase 6 (merge)
- **Claude:** Phase 2 (diagnosis), Phase 3 (plan), Phase 5 (review), Phase 7 (follow-up)
- **OpenCode:** Phase 4 (implementation)

### Q: What if I disagree with Claude's diagnosis in Phase 2?

**A:** Challenge it. Ask Claude to:
- Show evidence for the diagnosis
- Rule out other hypotheses
- Consider alternatives

If you're not convinced, don't proceed. Better to re-diagnose than to implement the wrong fix.

### Q: Can I run multiple tasks in parallel?

**A:** Yes, but each task must independently follow all 7 phases.

You can have Task A in Phase 4, Task B in Phase 5, Task C in Phase 1 — all at the same time. Each one stays in its own phase-gate sequence.

---

## Integrations

### Q: Can I automate this workflow?

**A:** Partially.

- **GitHub Actions** can trigger OpenCode runs on pull requests
- **Slack** can post issue summaries and review results
- **Linear** can auto-create follow-up tickets from `$to-tickets`

No complete automation yet — the phases still require human decisions.

### Q: Can I use this workflow with other tools (Jira, GitLab, etc.)?

**A:** Yes. The workflow is tool-agnostic.

- Issues can live in GitHub, Linear, Jira, or local ISSUES.md
- Delegation can use OpenCode, or any other code-generation tool
- Review is independent of any tool

The principles apply everywhere.

---

## Troubleshooting

### Q: My team is resisting the workflow. It feels like bureaucracy.

**A:** Process does add overhead. The argument for it:

1. **Mistakes are expensive** — phase gates catch them early
2. **Audit trail** — you can explain why any decision was made
3. **Consistency** — everyone uses the same approach

If your team is small and high-trust, you can be faster without process. But as projects grow, this kind of structure pays for itself.

### Q: How do I onboard a new team member?

**A:** 

1. Point them to [README.md](README.md)
2. Have them complete [SETUP.md](SETUP.md)
3. Walk them through [WORKFLOW.md](WORKFLOW.md)
4. Show them [CASE-STUDY.md](CASE-STUDY.md) — seeing a real example helps

Then pair on their first issue (Phases 1-3) so they see the flow in action.

### Q: What if I think a phase is unnecessary for my project?

**A:** Try it for 2 weeks as written. Then evaluate.

The phases are structured for a reason. Most shortcuts create problems later. If you still think a phase is wasteful after two weeks, we can discuss customization.

---

## Getting Help

- **General questions?** → Ask in this repo's Issues
- **Workflow question?** → Re-read [WORKFLOW.md](WORKFLOW.md)
- **Setup trouble?** → See [SETUP.md#troubleshooting](SETUP.md#troubleshooting)
- **Real-world example?** → Read [CASE-STUDY.md](CASE-STUDY.md)

---

**Last updated:** 2026-09-22  
**Maintained by:** Ibrahim's team

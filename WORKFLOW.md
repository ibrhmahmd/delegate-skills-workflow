# The 7-Phase Workflow — Detailed

Every task, no matter how small, follows these 7 phases. No exceptions, no skipping.

---

## Phase 1: Issue Creation

**Owner:** You  
**Time:** 5-10 minutes  
**Outcome:** A tracking issue with clear description and phase tag

### What to do

Create a tracking issue (GitHub Issues, Linear, or local `ISSUES.md`) with:

1. **Title** — one-line summary of what's being fixed/built
2. **Description** — 2-3 paragraphs explaining:
   - What's wrong (or what we're building)
   - Why it matters
   - Any relevant context (links to docs, past incidents, etc.)
3. **Type tag** — one of:
   - `feature` — new capability
   - `bug` — something broken
   - `refactor` — improving existing code
   - `docs` — documentation
   - `performance` — speed/efficiency improvement
4. **Lane assignment** — which lane owns this (feature/refactor/ui/docs/performance)

### Why this matters

**Without a tracking issue, you have no audit trail.** If someone asks "why did we do this?" three months later, the commit message alone won't tell the full story. The issue captures the decision, the diagnosis, and the approval.

### Example

See [examples/ISSUE_TEMPLATE.md](examples/ISSUE_TEMPLATE.md) for a template.

---

## Phase 2: Analysis & Planning

**Owner:** Claude  
**Time:** 15-45 minutes (depends on complexity)  
**Outcome:** Diagnosis of the actual problem (not assumptions)

### What to do

Use Claude Code skills to **understand the problem before proposing a solution:**

- **`$domain-modeling`** — if the issue touches business logic
- **`$codebase-design`** — if you need to understand the current architecture
- **`$wait-what`** — if the issue description is ambiguous or contradictory
- **`$diagnosing-bugs`** — if this is a production bug (error logs, symptoms, etc.)

### Key principle: Diagnosis first

**Don't start writing code until you've confirmed the actual root cause.** Many bugs have surface symptoms that look like one thing but are caused by something else entirely. A 30-minute diagnosis can save hours of wasted implementation.

### What Claude will deliver

A clear analysis document with:
- Root cause identified (with evidence)
- What's NOT the problem (wrong hypotheses ruled out)
- Scope of impact
- Recommended approach (not yet detailed code, just direction)

---

## Phase 3: Implementation Planning

**Owner:** Claude + You (discussion required)  
**Time:** 20-60 minutes  
**Outcome:** A detailed, approved plan ready for delegation

### What to do

Claude proposes a plan using:

- **`$request-refactor-plan`** — for refactors
- **`$improve-codebase-architecture`** — for architectural changes
- **`$tdd`** — for complex new features (test-first approach)

### Critical: Discuss the plan with the user

**Never delegate without approval.** The plan is not a surprise — Claude presents it to you, you ask questions, you approve it or ask for changes.

### What the plan should include

- Exact files to change (with line numbers)
- New functions/methods (with signatures)
- Testing strategy
- Deployment considerations (breaking changes? migrations? rollback plan?)
- Estimated effort

---

## Phase 4: Delegate to OpenCode

**Owner:** OpenCode (implementer)  
**Time:** varies (10 minutes to hours, depending on scope)  
**Outcome:** Code changes, uncommitted and ready for review

### What to do

Use the appropriate delegate skill:

- **`$feature-delegate`** — new features
- **`$refactor-delegate`** — refactoring
- **`$ui-delegate`** — frontend changes
- **`$docs-delegate`** — documentation
- **`$performance-delegate`** — performance improvements

Or invoke directly via shell:

```bash
opencode run --model opencode/grok-4.6 --agent build \
  --file /path/to/implementation-brief.md
```

### Critical: The implementation brief

**Write the prompt to a file and pass it with `--file`.** Do NOT inline it in double quotes on the command line — backticks, `$(...)`, and parens will get interpreted by the shell and mangle the prompt.

Example brief template:

```markdown
# Implementation Brief

## Task
Fix the daily report scheduler to use external cron instead of in-process loops.

## Files to change
- app/api/routers/notifications/internal_scheduler_router.py (new file)
- app/api/main.py (remove lifespan scheduler tasks)
- app/core/config.py (add INTERNAL_TRIGGER_SECRET setting)

## Specific changes required
1. Create 3 new endpoints: POST /internal/reports/{daily,weekly,monthly}-report/trigger
2. Each endpoint validates X-Internal-Trigger-Secret header
3. Each endpoint checks notification_logs for existing SENT row to prevent double-sends
4. Delete report_scheduler.py and report_watchdog.py (no longer needed)

## Tests expected
- Regression tests for auth validation
- Idempotency guard tests

## Do not commit. Changes remain uncommitted for review.
```

### OpenCode's responsibility

- Implement the changes
- Make tests pass
- Leave code **uncommitted** (Phase 5 review happens first)

---

## Phase 5: Code Review

**Owner:** Claude + You (you verify, Claude analyzes)  
**Time:** 30 minutes to 2 hours (real work, not a rubber-stamp)  
**Outcome:** Approved or re-delegated (with fixes)

### What to do

This is the real quality gate. Don't skip it or rush it.

1. **Run tests yourself** — don't just trust the implementer's self-report
   ```bash
   npm test
   # or
   pytest tests/
   ```

2. **Review the diff** — use `$code-review` for a professional review
   ```bash
   $code-review
   ```

3. **Grill complex sections** — use `$grill-me` to stress-test assumptions
   ```bash
   $grill-me "Does this handle concurrent writes safely?"
   ```

4. **Check for regressions** — if tests existed before, do they still pass?
   - If a test fails, verify it's not pre-existing: `git stash` → run test against original code
   - If it fails on original too, it's pre-existing (not your regression)

5. **Verify database safety** — if the code touches databases:
   - Test files that INSERT/COMMIT rows must NEVER run against production
   - Spin up a local Postgres if needed, or use an existing test database
   - Read-only checks against production are OK only with explicit user approval

### What to report

Report every finding with **file:line precision**. Example:

```
File: app/api/routers/notifications/internal_scheduler_router.py:50
Issue: _get_yesterday_cairo() uses date.today() (system timezone) instead of datetime.now(CAIRO_TZ)
Impact: Function doesn't actually use Cairo timezone despite its purpose
Fix: Change to datetime.now(CAIRO_TZ).date() - timedelta(days=1)
```

### If bugs are found

**Re-delegate with a precise fix brief** (same discipline as Phase 4). Don't fix small things directly — send them back to OpenCode for a second implementation round. This keeps the workflow clean: only Claude and OpenCode touch the code, you review both rounds.

---

## Phase 6: Merge & Close

**Owner:** You  
**Time:** 5 minutes  
**Outcome:** Code lands, issue closes

### What to do

1. **Get approval** — ask the user: "Ready to merge?"
2. **Land the code**:
   ```bash
   git add .
   git commit -m "Commit message describing the change

   closes #123
   Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
   git push origin main
   ```
3. **Close the issue** — the `closes #123` in the commit message does this automatically

### Commit message format

- **First line:** one-line summary (what changed)
- **Body:** 1-2 sentences explaining why
- **Footer:** `closes #N` (links to the tracking issue)
- **Attribution:** `Co-Authored-By: Claude <...>` (credits the AI)

Example:

```
Fix: scheduled daily report now uses external cron triggers instead of unreliable in-process loops

The in-process asyncio scheduler could not reliably fire at exact clock times on FastAPI Cloud's potentially-suspended containers. Replaced with HTTP trigger endpoints called by external cron.

closes #42
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

---

## Phase 7: Tracking & Follow-up

**Owner:** Claude  
**Time:** 10-15 minutes  
**Outcome:** Future work captured, team stays unblocked

### What to do

After landing, use `$to-tickets` to capture any:

- **Operational work** (credential rotation, cron setup, etc.) that couldn't be code-delegated
- **Refactoring debt** discovered during implementation
- **Monitoring/alerting** needed for this change
- **Documentation** that should be updated

Example:

```
$to-tickets
- Rotate Gmail App Password (Gmail dashboard, user-only step)
- Wire FastAPI Cloud scheduled jobs for daily/weekly/monthly report triggers
- Add logfire alerts for SMTP auth failures
```

These become new tracking issues for the team to handle.

---

## Skills by Phase

| Phase | Skills |
|-------|--------|
| **1. Issue Creation** | (none — just write clear issue text) |
| **2. Analysis** | `$domain-modeling`, `$codebase-design`, `$wait-what`, `$diagnosing-bugs` |
| **3. Planning** | `$request-refactor-plan`, `$improve-codebase-architecture`, `$tdd` |
| **4. Delegate** | `$feature-delegate`, `$refactor-delegate`, `$ui-delegate`, `$docs-delegate`, `$performance-delegate` |
| **5. Review** | `$code-review`, `$grill-me`, `$simplify` |
| **6. Merge** | (none — just git) |
| **7. Follow-up** | `$to-tickets`, `$adr-author`, `$handoff` |

---

## Common Questions

**Q: Can I skip Phase 2 if it's obvious what the problem is?**  
A: No. Even "obvious" bugs often have surprising root causes. A 20-minute diagnosis prevents a 2-hour rabbit hole.

**Q: What if the plan in Phase 3 is bad?**  
A: Push back. Discuss with Claude until you're confident. The plan gets re-done if needed. Better to debate on paper than to discover issues in Phase 5.

**Q: Can I do the code review quickly?**  
A: Not if "quickly" means skipping tests. Actually run the tests. Actually think about the code. Phase 5 is the quality gate — it's worth the time.

**Q: What if Phase 4 produces code I don't like stylistically?**  
A: As long as it's correct, passes tests, and follows the language/framework conventions — land it. Don't nitpick style in review.

---

**Next:** [SETUP.md](SETUP.md) for installation, or [CASE-STUDY.md](CASE-STUDY.md) for a real example.

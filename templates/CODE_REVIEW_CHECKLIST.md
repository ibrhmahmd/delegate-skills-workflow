# Code Review Checklist

Use this during Phase 5 to ensure thorough review. Check off items as you verify them.

---

## Correctness

- [ ] **Tests run independently** — I ran them myself (not just trusting the implementer's report)
  - Command: `[paste the command you ran]`
  - Result: `[X/Y tests passed]`
- [ ] **All new code is tested** — there's a test for every new function/endpoint
- [ ] **Edge cases are covered** — null inputs, empty arrays, negative numbers, boundary conditions
- [ ] **Error handling is present** — exceptions are caught and handled appropriately
- [ ] **Async/concurrency is handled** — if code uses async/threads, critical sections are protected

## Regressions

- [ ] **Existing tests still pass** — ran full test suite, no new failures
- [ ] **Pre-existing failures ruled out** — verified with `git stash` that failures existed on original code
- [ ] **Database impacts** — if this touches DB, queries are correct and efficient
- [ ] **Configuration** — if new env vars added, they're documented and validated

## Code Quality

- [ ] **No obvious bugs** — used `$code-review` to scan for issues
- [ ] **Performance is acceptable** — no O(n²) loops, no N+1 queries, no blocking operations
- [ ] **Follows project style** — matches language/framework conventions (linting passes)
- [ ] **No dead code** — all new code is actually used
- [ ] **No secrets** — no API keys, passwords, or credentials in code

## Security

- [ ] **Input validation** — user input is validated before use (especially in web endpoints)
- [ ] **No SQL injection** — if using raw SQL, it's parameterized
- [ ] **No XSS** — if output goes to HTML, it's escaped
- [ ] **No CSRF** — if modifying state, CSRF tokens/same-site checks are present
- [ ] **No privilege escalation** — auth checks are in place where needed

## Documentation

- [ ] **Code comments** — non-obvious logic has comments explaining the why, not the what
- [ ] **Function docstrings** — if public API, functions have docstrings
- [ ] **README updated** — if this adds user-facing features, docs are updated
- [ ] **Commit message** — descriptive, explains why not just what

## Deployment

- [ ] **No breaking changes** — unless intentional and documented
- [ ] **Database migrations** — if schema changes, migrations are present and tested
- [ ] **Rollback plan** — if this breaks, we can undo it
- [ ] **Monitoring** — if critical code, error logging is in place

## Complex Sections (if applicable)

Use `$grill-me` to stress-test these:

- [ ] **Concurrency/locking** — `$grill-me "Is this thread-safe?"` → [response summary]
- [ ] **Business logic** — `$grill-me "What if [edge case]?"` → [response summary]
- [ ] **Architecture** — `$grill-me "Is this the right approach?"` → [response summary]

---

## Findings Summary

| Issue | Severity | File:Line | Recommendation |
|-------|----------|-----------|---|
| [Issue 1] | Critical / Major / Minor | file.py:42 | Fix before landing / Can land as-is / Defer to follow-up |
| [Issue 2] | | | |

---

## Decision

- [ ] ✅ **Approve** — Ready to merge
- [ ] 🟡 **Re-delegate** — Bugs found, send back to implementer with fix brief
- [ ] 🔴 **Block** — Major issues, needs discussion before proceeding

**Approval:** [Your name] on [Date]

**Notes:**  
[Any additional comments or context]

---

## Re-Delegation (if needed)

If you found bugs, re-delegate using [templates/IMPLEMENTATION_BRIEF.md](IMPLEMENTATION_BRIEF.md):

```markdown
# Fix Brief: [Issue Title]

## Bugs found in review

1. File: app/api/routers/notifications/internal_scheduler_router.py:50
   Issue: _get_yesterday_cairo() uses date.today() instead of datetime.now(CAIRO_TZ)
   Fix: Change to datetime.now(CAIRO_TZ).date() - timedelta(days=1)

2. [Bug 2]
3. [Bug 3]

After fixing, run tests again and confirm all pass before finishing.
```

---

**See also:** [WORKFLOW.md#phase-5--code-review](../WORKFLOW.md#phase-5--code-review) for details.

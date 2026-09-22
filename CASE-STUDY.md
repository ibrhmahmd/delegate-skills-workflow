# Case Study: Techno-Terminal Daily Report Bug

Real-world example of the 7-phase workflow applied to a production bug.

**Project:** Techno-Terminal (school management SaaS)  
**Bug:** Scheduled daily report emails stopped sending (15 days of silence)  
**Resolution time:** 2 days (diagnosis + implementation + testing)  
**Phases completed:** All 7, with complications and re-delegation

---

## The Problem (Phase 1: Issue Creation)

**Issue #1: Scheduled daily report email does not send**

- **Status:** 🔴 Critical (revenue-blocking — admins monitor this daily)
- **Type:** bug
- **Lane:** refactor (architectural fix, not a feature)
- **Created:** 2026-09-19

### Description

> The scheduled daily business report email never arrives. Other notification emails (payment receipts, enrollment confirmations) work fine, so the email dispatch mechanism itself is not broken — the problem is scoped to the daily report path specifically.
>
> Last successful send: 2026-09-04 (15 days ago)  
> Current status: zero log entries in `notification_logs` for `daily_report` (not even FAILED rows)

---

## Phase 2: Analysis & Planning

**Key question:** Why would the send loop produce zero logs?

### Initial Hypothesis (Wrong)

"Recipients or template are misconfigured."

**Verification:** Queried `admin_notification_settings` against production database → found that daily_report is correctly enabled, template exists and is active. ❌ Not the cause.

### Root Cause Analysis (Correct)

**Used:** `$diagnosing-bugs` skill to trace the code path:

1. Looked at `app/modules/notifications/services/report_scheduler.py` → found `start_report_scheduler()` is an in-process `while True: await asyncio.sleep(60)` loop
2. Loop only fires if process is alive at exact scheduled time (1:31 AM Cairo) — a dead-traffic hour
3. Project is hosted on **FastAPI Cloud** (mentioned in git history: Aug 31 commit literally says "masking the post-FastAPI-Cloud report outage")
4. **Conclusion:** FastAPI Cloud suspends idle containers with no inbound traffic. The scheduler loop never gets CPU time at the trigger minute, so it never fires.

### Evidence

- Aug 31 commit `c9ca5fa`: "make scheduled report failures visible via logfire reorder, watchdog alert, and regression test" — contains the phrase "masking the post-FastAPI-Cloud report outage", confirming this exact failure happened before
- Last successful send: 2026-09-04 (before the container suspension pattern became consistent)
- Zero log rows since (not even FAILED — the loop never started)

### Secondary Bug Discovered

During diagnosis, found Gmail SMTP auth is failing (`535 Bad Credentials`) since 2026-09-18, affecting payment_receipt and enrollment_confirmation emails too. Scoped into same issue per user decision.

---

## Phase 3: Implementation Planning

**User decision:** Fix both bugs in one ticket.

### Bug #1 Solution: External Cron + HTTP Triggers

Instead of in-process scheduler (unreliable on this host):

1. Add 3 authenticated HTTP trigger endpoints:
   - `POST /internal/reports/daily-report/trigger`
   - `POST /internal/reports/weekly-report/trigger`
   - `POST /internal/reports/monthly-report/trigger`
   - Protected by `X-Internal-Trigger-Secret` header
2. DB-backed idempotency check (query `notification_logs` for existing SENT row)
3. Delete the dead scheduler + watchdog loops
4. Add regression tests
5. **Operational step (human-only):** Wire external cron to call endpoints

### Bug #2 Solution: Loud Error Logging

Make SMTP auth failures emit a distinct `logfire.error("smtp_auth_failed", ...)` event so they're visible and alertable (not buried in generic send-failure logs).

### Plan Discussion

Claude proposed this architecture, user approved it. Changes were precise and scoped.

---

## Phase 4: Delegation to OpenCode

**Model:** `opencode/grok-4.6`  
**Scope:** Code changes only (operational steps deferred to later)

### Implementation Brief (Sent to OpenCode)

```
# Bug Fix Brief

Fix Issue #1: Scheduled daily report not sending
Root cause: in-process asyncio scheduler cannot reliably fire at fixed times 
on FastAPI Cloud (container suspension). Replacement with HTTP endpoints.

## Files to create/modify:
1. app/api/routers/notifications/internal_scheduler_router.py (NEW)
   - 3 endpoints for daily/weekly/monthly report triggers
   - Auth via X-Internal-Trigger-Secret header
   - DB-backed idempotency (check notification_logs for existing SENT)

2. app/api/main.py (MODIFY)
   - Remove start_report_scheduler() and start_report_watchdog() from lifespan
   - Remove imports for deleted modules

3. app/core/config.py (MODIFY)
   - Add internal_trigger_secret: str setting

4. app/modules/notifications/dispatchers/email_dispatcher.py (MODIFY)
   - Add logfire.error("smtp_auth_failed") on SMTP auth errors

5. Delete:
   - app/modules/notifications/services/report_scheduler.py
   - app/modules/notifications/services/report_watchdog.py

## Tests
- tests/test_internal_scheduler.py (new file)
- Should test: auth validation, idempotency, endpoint behavior

## Do not commit. Changes remain uncommitted for review.
```

**Result:** OpenCode delivered working code in ~90 minutes, but with 5 bugs found in review (see Phase 5).

---

## Phase 5: Code Review

**Approach:** Independent test execution + detailed review

### Test Environment

Testing DB was unreachable (Supabase testing project paused). Solution: Spun up local Postgres with schema + migrations, ran tests against it.

### Bugs Found & Re-Delegated

| # | Bug | Severity | Fix |
|---|---|---|---|
| 1 | Missing header returned 422 instead of 401 | Real | Declare header as `Optional[str] = Header(None)`, handle None explicitly |
| 2 | `_get_yesterday_cairo()` used system tz not Cairo | Real | Use `datetime.now(CAIRO_TZ).date()` instead of `date.today()` |
| 3 | 4 test methods missing `client` fixture parameter | Test-only | Add `client` to test signatures |
| 4 | Helper `_create_sent_log` in wrong class | Test-only | Move to module level |
| 5 | `uv.lock` added as artifact | Minor | Flag for commit decision |

### Re-Delegation Brief

Sent precise, file-line-referenced brief back to OpenCode with all 5 issues. Used a **file-based prompt** (`--file`) to avoid shell-quoting bugs.

**Result after re-delegation:** All tests pass (13/13 in test_internal_scheduler.py).

### Regression Check

Ran `test_notifications.py` and `test_notifications_full.py` to check for regressions from deleting scheduler/watchdog files and modifying main.py, config.py, email_dispatcher.py.

Found 8 test failures, but verified via `git stash` against the original code — **all 8 failures exist on unmodified code**, so they're pre-existing (unrelated to this work).

---

## Phase 6: Merge & Close

**Commit message:**

```
fix: replace in-process scheduler with external HTTP triggers + loud SMTP logging

The in-process asyncio daily/weekly/monthly report schedulers could not 
reliably fire at exact clock times on FastAPI Cloud's potentially-suspended 
containers. Replaced with authenticated HTTP trigger endpoints 
(/internal/reports/{daily,weekly,monthly}-report/trigger) called by 
external cron.

Added DB-backed idempotency guard (checks notification_logs) to prevent 
duplicate sends with multiple workers.

Also made SMTP auth failures escalate loudly via logfire.error("smtp_auth_failed") 
so production SMTP issues (like the current 535 Bad Credentials) don't get 
silently buried.

Deleted: report_scheduler.py, report_watchdog.py (no longer needed)
Tested: 13/13 regression tests pass, pre-existing test failures unrelated

closes #1
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

---

## Phase 7: Follow-up

**Operational work captured via `$to-tickets`:**

1. **Rotate Gmail App Password** (dashboard user-only step, not code)
2. **Wire FastAPI Cloud scheduled jobs** to call the 3 trigger endpoints daily
3. **Add logfire alerts** for SMTP auth failures so this doesn't go dark again

---

## Key Takeaways

### What Went Well

✅ **Phase 2 diagnosis was thorough** — ruled out wrong hypotheses, found the real root cause  
✅ **Phase 3 plan was discussed before delegating** — no surprises later  
✅ **Phase 5 review caught real bugs** — the 422 vs 401 bug would have been a support issue  
✅ **Re-delegation was precise** — bugs were fixed in one round  
✅ **Tests were run independently** — not just trusting OpenCode's self-report  

### What Could Improve

⚠️ **Phase 4 initial implementation had 5 bugs** — better code quality from OpenCode might reduce re-delegation needs  
⚠️ **Operational steps (Gmail, cron wiring) still pending** — not yet done as of 2026-09-22  
⚠️ **Testing DB was unreachable** — had to set up local Postgres, adding friction

---

## Timeline

| Date | Phase | Status |
|------|-------|--------|
| 2026-09-19 | 1-2 | Issue created, root cause diagnosed |
| 2026-09-19 | 3-4 | Plan discussed, OpenCode delegation completed |
| 2026-09-19 | 5 | Review found 5 bugs, re-delegation issued |
| 2026-09-19 | 5 | Re-delegation completed, all tests pass |
| 2026-09-19 | 6 | Code landed, issue closed |
| 2026-09-22 | 7 | Follow-up tickets captured (Gmail/cron still pending) |

**Total development time:** ~2 days (mostly waiting on network, not coding)

---

## Conclusion

This case study shows the workflow handling a **real production bug** with:
- Proper diagnosis (not guessing)
- Scoped delegation (precise briefs)
- Rigorous review (independent test execution)
- Re-delegation when bugs are found
- Tracking of operational follow-up

The overhead of the 7 phases was ~3-4 hours total. Without it, the bug fix might have been quicker to write, but the lack of process discipline would have meant:
- Risk of landing wrong changes (Phase 5 catches this)
- No audit trail of why decisions were made
- Higher chance of similar bugs in the future (no pattern recognition)

**Worth the process overhead.**

---

**See also:** [WORKFLOW.md](WORKFLOW.md) for the full phase-by-phase breakdown.

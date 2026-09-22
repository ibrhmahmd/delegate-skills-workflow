# Implementation Brief Template

Send this to OpenCode in Phase 4. Write this file and pass it with `opencode run --file brief.md`.

---

# Implementation Brief: [Task Title]

## Summary

One paragraph describing what you're implementing and why.

Example:

> Fix Issue #42: Replace the in-process scheduler with external HTTP triggers.
> The in-process asyncio loop cannot reliably fire at fixed times on FastAPI Cloud
> (container suspension). Solution: authenticated HTTP endpoints called by external cron.

## What to change

List exact files, with line numbers/functions where relevant.

```
- app/api/routers/notifications/internal_scheduler_router.py (NEW FILE)
  * Create 3 new endpoints: POST /internal/reports/{daily,weekly,monthly}-report/trigger
  * Each endpoint validates X-Internal-Trigger-Secret header
  * Each endpoint checks notification_logs for existing SENT row to prevent double-sends

- app/api/main.py (MODIFY)
  * Lines 45-67: Remove start_report_scheduler() and start_report_watchdog() from lifespan
  * Lines 12-15: Remove imports for report_scheduler and report_watchdog

- app/core/config.py (MODIFY)
  * Add setting: internal_trigger_secret: str = "" (empty by default, rejecting all)

- app/modules/notifications/dispatchers/email_dispatcher.py (MODIFY)
  * Lines 89-92: On SMTPAuthenticationError, emit logfire.error("smtp_auth_failed", recipient=recipient)

- DELETE:
  * app/modules/notifications/services/report_scheduler.py (no longer needed)
  * app/modules/notifications/services/report_watchdog.py (replaced by DB-backed idempotency)

- tests/test_internal_scheduler.py (NEW FILE)
  * Regression tests for auth validation
  * Regression tests for idempotency guard
```

## Key requirements

Things that MUST be in the implementation:

- ✅ All 3 endpoints return 401 (not 422) on auth failure
- ✅ Idempotency check queries `notification_logs` for existing SENT row
- ✅ Tests pass locally (verify yourself: `pytest tests/test_internal_scheduler.py -v`)
- ✅ No new dependencies added
- ✅ Code follows project style (linting passes)

## Testing strategy

How to verify this works:

1. Run regression tests: `pytest tests/test_internal_scheduler.py -v`
2. Manual auth test:
   - Call endpoint with missing header → expect 401
   - Call endpoint with wrong secret → expect 401
   - Call endpoint with correct secret → expect 200
3. Idempotency test:
   - Call endpoint once → sends email, returns success
   - Call again immediately → returns "already sent today", skips send
4. Verify existing tests still pass: `pytest tests/test_notifications_full.py -v`

## Deployment notes

Before this ships to production:

- User must set `INTERNAL_TRIGGER_SECRET` env var (or all requests will be rejected with 401)
- User must wire external cron to call the new endpoints
- No breaking changes to existing API

## Do not

- ❌ Don't commit these changes (leave uncommitted for Phase 5 review)
- ❌ Don't add new dependencies
- ❌ Don't change database schema
- ❌ Don't modify the .env or config files beyond what's listed above
- ❌ Don't skip tests

---

## Related Issues

- Issue #42 (this one)
- Issue #43 (Gmail SMTP auth — separate brief)

## Questions?

If anything is unclear, ask before starting implementation.

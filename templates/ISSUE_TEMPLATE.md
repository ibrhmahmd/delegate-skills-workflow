# Issue Template

Copy this template for every new issue. Fill in the sections, commit to your ISSUES.md (or GitHub Issues).

---

## #N — [One-line description]

**Status:** 🟡 IN PROGRESS (or 🔴 BLOCKED, 🟢 DONE)  
**Type:** `bug` / `feature` / `refactor` / `docs` / `performance`  
**Lane:** `feature` / `refactor` / `ui` / `docs` / `performance`  
**Created:** YYYY-MM-DD  
**Owner:** @username (or leave blank if unassigned)

### Description

2-3 paragraphs explaining:
- What's broken or what we're building
- Why it matters
- Relevant context (links, past incidents, etc.)

Example:

> The scheduled daily report email never arrives. Other notification emails
> (payment receipts, enrollment confirmations) work fine, so the email dispatch
> mechanism itself is not broken — the problem is scoped to the daily report
> path specifically.
>
> Last successful send: 2026-09-04 (15 days ago)  
> Current status: zero log entries in `notification_logs`

### Phase 2: Diagnosis

(Filled in after Claude's analysis)

**Root Cause:**  
[Brief summary of what's actually wrong, with evidence]

**Impact:**  
[What breaks if this isn't fixed]

**Ruling Out Hypotheses:**  
- ❌ [Wrong hypothesis 1 — why it's not this]
- ❌ [Wrong hypothesis 2 — why it's not this]

### Phase 3: Implementation Plan

(Filled in after planning discussion)

**Approach:**  
[High-level summary of the fix strategy]

**Files to Change:**  
- `app/file1.py` — what changes
- `app/file2.py` — what changes
- (NEW) `app/file3.py` — new file

**Estimated Effort:**  
[Hours for implementation + testing]

**Risks:**  
[Deployment concerns, breaking changes, rollback plan]

### Phase 4: Delegation

**Delegated to:** OpenCode  
**Status:** ⏳ In progress / ✅ Complete / ❌ Needs revision

**Changes:**  
[Link to PR or brief list of files changed]

### Phase 5: Review

**Reviewer:** Claude  
**Test Results:** ✅ All pass (13/13 tests_internal_scheduler.py)  
**Bugs Found:** None / [List of bugs found, if any]

**Approval:** 🟢 Ready to merge / 🟡 Re-delegation needed / 🔴 Blocked

### Phase 6: Merge

**Committed:** 2026-09-19 by @ibrahim  
**Commit:** `abc123f — fix: replace in-process scheduler with external HTTP triggers`

### Phase 7: Follow-up

**Operational Work:**  
- [ ] Rotate Gmail App Password (user step)
- [ ] Wire FastAPI Cloud scheduled jobs
- [ ] Add logfire alerts for SMTP failures

**Deferred Issues:**  
- [#2] Upgrade email templates (lower priority)
- [#3] Add calendar sync feature (future)

---

## Notes

- Keep this updated as the issue progresses through phases
- Link to external resources (Figma, Slack threads, etc.)
- Use the status emoji to scan at a glance: 🟢 (done), 🟡 (in progress), 🔴 (blocked/critical)

## Closes

[Reference the original tracking issue, if any]

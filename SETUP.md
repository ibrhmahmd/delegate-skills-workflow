# Setup Guide

Get delegate-skills installed and configured on your machine.

---

## Prerequisites

- **Claude Code CLI** installed (if not: `curl -fsSL https://claude.com/download | bash`)
- **Node.js 16+** (`node --version`)
- **git** installed
- A GitHub account (for syncing skills)

---

## Step 1: Install Skills (One-Time)

```bash
npx skills add amElnagdy/delegate-skills
```

This installs the delegate-skills collection globally at `~/.agents/skills/`.

### What you get

The installation includes:
- `delegate-setup` — initialize a new project
- `opencode-delegate` — hand off work to OpenCode
- `code-review` — review implementation (mattpocock)
- `domain-modeling` — understand the problem space
- `codebase-design` — explore architecture
- `tdd` — test-first design
- `to-tickets` — capture follow-up work
- `grilling` — stress-test ideas
- `diagnosing-bugs` — deep-dive debugging
- And 10+ more

Run `npx skills list` to see the full list.

---

## Step 2: Copy Fleet Configuration

Copy the default fleet config to your home directory:

```bash
cp examples/delegate-fleet.json ~/.config/delegate-fleet.json
```

This defines 5 lanes (feature, refactor, ui, docs, performance), all bound to OpenCode with the `grok-4.6` model.

### Customize if needed

Edit `~/.config/delegate-fleet.json` to:
- Change the OpenCode model (if a newer one is available)
- Add new lanes
- Assign different implementers to different lanes

Example:

```json
{
  "version": "delegate-fleet.v1",
  "lanes": {
    "feature": { "implementer": "opencode", "model": "opencode/grok-4.6" },
    "refactor": { "implementer": "opencode", "model": "opencode/grok-4.6" },
    "ui": { "implementer": "opencode", "model": "opencode/grok-4.6" },
    "docs": { "implementer": "claude", "model": "claude-sonnet-5" },
    "performance": { "implementer": "opencode", "model": "opencode/grok-4.6" }
  }
}
```

---

## Step 3: Copy Global Instructions

Copy the mandatory workflow rules to your home directory so every Claude Code session loads them:

```bash
cp examples/CLAUDE.md ~/.claude/CLAUDE.md
```

This file is loaded automatically in every project, enforcing the 7-phase workflow globally.

### Customize if needed

Edit `~/.claude/CLAUDE.md` to:
- Add project-specific rules
- Reference internal tools or dashboards
- Customize skill recommendations

---

## Step 4: Verify Installation

Check that everything is wired up:

```bash
# List installed skills
npx skills list | grep -i delegate

# Check fleet config
cat ~/.config/delegate-fleet.json

# Check global instructions
cat ~/.claude/CLAUDE.md | head -20
```

---

## Step 5: Set Up a New Project

In your new project repo:

```bash
# Initialize
git init
git remote add origin <your-repo-url>

# Copy project-specific templates
cp -r delegate-skills-workflow/templates .claude/

# Create an ISSUES.md for local issue tracking (if not using GitHub)
touch ISSUES.md

# Commit
git add .
git commit -m "chore: initialize delegate-skills workflow"
```

That's it. The global `~/.claude/CLAUDE.md` applies automatically in every session.

---

## Troubleshooting

### Skills not found

If `npx skills add` fails with "not found":

1. Check GitHub status (`github.com/status`)
2. Verify `~/.agents/skills/` directory exists
3. Try again: `npx skills add amElnagdy/delegate-skills --force`

### Fleet config not loading

If OpenCode doesn't recognize your lanes:

1. Check the path: `cat ~/.config/delegate-fleet.json`
2. Validate JSON: `python3 -m json.tool ~/.config/delegate-fleet.json`
3. Check format matches the example in this file

### Global CLAUDE.md not loading

If the 7-phase workflow isn't being enforced in a new session:

1. Check the file exists: `ls -la ~/.claude/CLAUDE.md`
2. Check content: `head -20 ~/.claude/CLAUDE.md`
3. Restart Claude Code CLI: `pkill -f 'claude code'` then start a new session

---

## Next Steps

1. **Read the workflow:** [WORKFLOW.md](WORKFLOW.md)
2. **See a real example:** [CASE-STUDY.md](CASE-STUDY.md)
3. **Create your first issue** following [Phase 1](WORKFLOW.md#phase-1--issue-creation)

---

## Optional: Advanced Configuration

### Custom Skills

To add your own organization-specific skills:

1. Create a skills repo: `github.com/your-org/your-skills`
2. Add it: `npx skills add your-org/your-skills`
3. Reference in CLAUDE.md: `Use $your-skill for X`

### Integrations

- **GitHub Actions** — trigger `opencode run` from CI/CD
- **Slack** — post issue links and review summaries
- **Linear/Jira** — sync issues with automated creation

See [FAQ.md](FAQ.md#integrations) for examples.

---

**Next:** [WORKFLOW.md](WORKFLOW.md) or [CASE-STUDY.md](CASE-STUDY.md)

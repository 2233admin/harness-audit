---
name: harness-audit
description: Use when Claude Code harness has grown over time and needs dead weight identification, or after a major model upgrade when scaffolding built to compensate for old model limitations may now be obsolete.
---

# Harness Audit

## Overview

Every hook, MCP, and skill encodes an assumption about what Claude can't do on its own. Those assumptions go stale as the model improves. This skill audits your harness, annotates each component with its origin purpose, and flags dead weight.

**Core principle:** Build to delete. Every harness component needs a birth certificate and an expiration check.

## When to Use

- After a major Claude model version upgrade
- Harness has grown organically for months with no cleanup
- Unexpected slowdowns, hook failures, or conflicts
- `budget-savings.log` / similar telemetry shows a hook never fires

## Workflow

### Phase 1: Inventory

```bash
cat ~/.claude/settings.json
cat ~/.claude/settings.local.json
cat .claude/settings.json 2>/dev/null   # project-level
ls ~/.claude/skills/                     # active skills
```

Catalog all: Stop/PreToolUse/PostToolUse/SessionStart hooks · mcpServers · enabledPlugins · active skills.

### Phase 2: Annotate

For each component, answer:

1. **What does it do?** (observable behavior)
2. **What model deficiency does it compensate for?**
3. **Is that deficiency still present?**

Write to `.claude/harness-manifest.md` (see format below).

### Phase 3: Classify

| Status | Criteria |
|--------|----------|
| **KEEP** | Deficiency still present, or compensates for environment (not model) |
| **REVIEW** | Partially superseded — test with/without |
| **DEAD WEIGHT** | Model now handles this natively |
| **UNKNOWN** | Origin unclear — investigate before any removal |

### Phase 4: Expiration Triggers

For each KEEP, record when to revisit:

```markdown
### hooks.Stop.world-model-snap.py
- Purpose: cross-session world state
- Compensates for: no native session persistence
- Status: KEEP
- Expiration trigger: when native session persistence ships
```

## Manifest Format

`.claude/harness-manifest.md`:

```markdown
# Harness Manifest
Generated: {date} | Model: {model-id}

## Dead Weight Candidates
- [ ] hooks.Stop.foo.py — [reason]
- [ ] mcpServers.puppeteer — redundant with chrome-in-chrome

## Components

### hooks.Stop.{script}
- **Purpose**: {what it does}
- **Compensates for**: {model deficiency or environment limit}
- **Status**: KEEP | REVIEW | DEAD WEIGHT | UNKNOWN
- **Expiration trigger**: {condition}
```

## Common Dead Weight Patterns

| Pattern | Signal |
|---------|--------|
| Context anxiety handler | Model no longer abandons tasks near context limit |
| Sprint decomposer | Model handles long tasks without forced checkpoints |
| Retry wrapper | Model recovers from tool failures natively |
| Duplicate memory | Two systems solving same problem |
| Output formatter | Model formats consistently without prompting |

## What NOT to Remove

- Environment limits (filesystem, network, OS) — won't improve with model upgrades
- Security boundaries and permission gates — intentional constraints, not compensations
- Observability (logging, backup) — serve you, not the model

## Before Deleting Anything

```bash
# 1. Disable first (comment out, don't delete)
# 2. Run 3+ representative sessions
# 3. Compare output quality and behavior
# 4. Clean → delete. Broken → re-enable, update status to KEEP.
```

Never remove based on manifest alone. Test the hypothesis.

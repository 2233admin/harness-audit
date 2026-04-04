# harness-audit

> Every hook, MCP, and skill in your Claude Code harness encodes an assumption about what Claude can't do on its own. Those assumptions go stale as the model improves.

A Claude Code skill that audits your harness for dead weight — annotating each component with its origin purpose, then flagging what the current model has outgrown.

## The Problem

You add a hook to fix a model limitation. The model improves. The hook stays. Six months later you have:

- A context-reset mechanism for "context anxiety" that Opus 4.5 no longer has
- A retry wrapper for tool failures the model now recovers from natively  
- Two session recorders solving the same problem
- Hooks that fire on every tool call but never actually do anything useful

This is **dead weight** — scaffolding that was once necessary, now just adding latency and complexity.

## Install

```bash
ccpi install 2233admin/harness-audit
```

Or manually copy `skills/harness-audit/SKILL.md` to `~/.claude/skills/harness-audit/SKILL.md`.

## Usage

Just describe what you want in Claude Code:

```
/harness-audit
```

Or naturally:

> "Audit my harness for dead weight"
> "Which of my hooks are still earning their keep?"
> "I just upgraded to Opus 4.6, what can I trim?"

Claude will:
1. Inventory all hooks, MCPs, plugins, and skills
2. Annotate each with its origin purpose and what model deficiency it compensates for
3. Classify: **KEEP** / **REVIEW** / **DEAD WEIGHT** / **UNKNOWN**
4. Write a manifest to `.claude/harness-manifest.md`
5. Give you a dead weight checklist to work through

## Output

`.claude/harness-manifest.md` — annotated inventory of your entire harness:

```markdown
# Harness Manifest
Generated: 2026-04-04 | Model: claude-sonnet-4-6

## Dead Weight Candidates
- [ ] hooks.Stop.context-reset.py — model no longer abandons tasks near context limit
- [ ] mcpServers.puppeteer — redundant with chrome-in-chrome

## Components

### hooks.Stop.session-to-chat.py
- **Purpose**: Save session conversation to disk
- **Compensates for**: No native session persistence
- **Status**: KEEP
- **Expiration trigger**: When Claude gains native cross-session memory
```

## Core Insight

From Anthropic's engineering blog:

> *"In a long-horizon agent, Sonnet 4.5 would abandon tasks near context limits. We added a context reset mechanism to compensate. With Opus 4.5, that behavior disappeared. The reset mechanism became dead weight."*

The pattern repeats with every model upgrade. **Build to delete.**

## What It Won't Touch

- Hooks compensating for **environment** limits (filesystem, OS, network) — these won't improve with model upgrades
- Security boundaries and permission gates — intentional constraints, not compensations  
- Observability (logging, backup) — serve you, not the model

## Contributing

Issues and PRs welcome. The skill is a single markdown file — easy to fork and adapt.

## License

MIT

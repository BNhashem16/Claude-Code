# SuperClaude as Default Tooling — Mandatory Rule

> When working in this repository with Claude Code (or any compatible AI agent), default to the **SuperClaude Framework** (https://github.com/SuperClaude-Org/SuperClaude_Framework) `/sc:*` commands and bundled agents for any task where one fits. Plain free-form prompting is the fallback, not the default.
>
> Applies to features, bug fixes, refactors, code review, planning, testing, docs — every AI-assisted change in this repo.

## Why

SuperClaude bundles structured personas, repo indexing, command-specific guardrails, and reviewer agents that produce more consistent, higher-quality output than free-form prompting. The four other rules in this folder (`phpstan-max`, `laravel-idiomatic`, `security-baseline`, `migrations-safety`, `api-contract`) define **what** good code looks like; SuperClaude is the **workflow** that gets us there reliably.

## Setup (once per machine)

```bash
# Install (Python 3.11+)
pip install SuperClaude

# Register slash commands and agents with Claude Code
SuperClaude install

# Verify
SuperClaude --version    # expect 4.2.0 or newer
SuperClaude doctor       # health check
```

If `/sc:<cmd>` ever misbehaves, run `SuperClaude update` to pull the latest commands.

## Default Command Routing

Always reach for the matching `/sc:*` command FIRST. Switch to free-form only when no SuperClaude command fits.

| Task type | Default command |
| --- | --- |
| Plan a new feature or non-trivial change | `/sc:brainstorm` → `/sc:design` → `/sc:workflow` |
| Implement a feature | `/sc:implement` |
| Improve / refactor existing code | `/sc:improve` |
| Analyze codebase or understand a module | `/sc:analyze` (run `/sc:index-repo` first on large repos) |
| Debug or root-cause a bug | `/sc:troubleshoot` |
| Write or update tests | `/sc:test` |
| Write or update documentation | `/sc:document` |
| Explain code to a teammate | `/sc:explain` |
| Estimate effort | `/sc:estimate` |
| Dead code / consolidation | `/sc:cleanup` |
| Git operations (branches, commits, PRs) | `/sc:git` |
| Task tracking within a session | `/sc:task` / `/sc:pm` |
| Save / resume a working session | `/sc:save` / `/sc:load` |
| Spawn a specialized reviewer or builder agent | `/sc:agent <name>` |
| Discover available commands | `/sc:recommend` or `/sc:help` |
| Spec review / panel critique | `/sc:spec-panel` / `/sc:business-panel` |

## Bundled Agents to Prefer

Use these via `/sc:agent <name>` or by direct subagent invocation:

- `planner` — implementation planning for non-trivial features
- `architect` — system design and architectural decisions
- `tdd-guide` — write tests first, implement to pass
- `code-reviewer` — general code quality review
- `security-reviewer` — OWASP / sensitive-data review
- `database-reviewer` — schema and query review (pairs well with `migrations-safety.md`)
- `refactor-cleaner` — dead code + duplication removal
- `performance-optimizer` — bottleneck and N+1 analysis (pairs with `api-contract.md`)
- `doc-updater` — keeps docs / codemaps in sync
- `build-error-resolver` — compile/build/type errors

Specifically for this Laravel stack, prefer:
- `code-reviewer` and `security-reviewer` on every PR.
- `database-reviewer` whenever a migration is touched.
- `performance-optimizer` whenever a query or response is changed.

## When NOT to use SuperClaude

- The user invoked a different specific tool / skill / plain prompt in the same message — respect their explicit choice.
- The task is a trivial single-tool call (one `Read`, one `grep`, one `Edit` for a typo). Don't add ceremony.
- A SuperClaude command itself is broken — fix or update it before using.

## Quick Self-Checklist

- [ ] Did I check whether a `/sc:*` command fits this task before writing a free-form prompt?
- [ ] For a non-trivial change: did I plan with `/sc:brainstorm` or `/sc:design` first?
- [ ] For implementation: am I using `/sc:implement` with the right context?
- [ ] For review: did I invoke `code-reviewer` / `security-reviewer` (and `database-reviewer` if migrations were touched)?
- [ ] For commits and PRs: did I use `/sc:git`?
- [ ] If SuperClaude misbehaves, did I run `SuperClaude update` and `SuperClaude doctor`?

> Treat this file as binding alongside the other rules in this directory. SuperClaude is the default workflow tooling — not optional.

# Minority Report

**Multi-agent PR review for Claude Code.** Three independent AI reviewers analyze your PR in parallel, then synthesize findings with consensus-based confidence scoring.

```
/mr
```

That's it. One command. Three reviewers. One report.

---

## How it works

Three **precogs** — independent AI review agents — each analyze your PR through a different lens:

| Precog | Lens | What they catch |
|--------|------|-----------------|
| **Agatha** | Security | Injection, auth bypass, data exposure, open redirects, info leaks |
| **Arthur** | Logic & Correctness | Race conditions, null handling, edge cases, missing tests, cross-repo impact |
| **Dash** | Architecture & Performance | Pattern violations, N+1 queries, dead code, missing migrations, resource leaks |

**Anderton** synthesizes their findings into a single report:

- **Previsions** — 2+ precogs flagged the same issue (high confidence)
- **Minority Reports** — only one precog saw it (lower confidence, but potentially the most important finding)
- **Confidence scoring** — each finding rated 1-10 so you know what's confirmed vs. speculative

> In the movie, the minority report was the dissenting precog who proved the consensus wrong.
> In code review, it's the single reviewer who catches what everyone else missed.

---

## Install

Copy `mr.md` to your Claude Code commands directory:

### Global (available in all projects)

```bash
# Create the directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy the command
cp mr.md ~/.claude/commands/mr.md
```

### Project-level (available only in one project)

```bash
# From your project root
mkdir -p .claude/commands

# Copy the command
cp mr.md .claude/commands/mr.md
```

That's it. No dependencies, no config, no API keys beyond what Claude Code already uses.

---

## Usage

```bash
# Review current branch vs main
/mr

# Review a specific branch
/mr feature-branch

# Review a GitHub PR by number
/mr 142
```

### What you get

```markdown
## Minority Report

**Branch**: feature-branch
**Files changed**: 5
**Lines**: +241 / -372
**Precogs**: Agatha (Security), Arthur (Logic), Dash (Architecture)

---

### Previsions — Consensus (2+ precogs agree)
> All precogs saw this future — highest confidence.

**[Agatha + Arthur] `str(e)` leaks internal details to API consumer** (confidence: 9/10)
...

### Minority Reports — Single Precog (confidence >= 7)
> Only one precog saw this — but they're certain.

**[Dash] Ad-hoc metadata keys bypass existing DegradationFlags pattern** (confidence: 8/10)
...

### Anderton's Recommendation: APPROVE WITH SUGGESTIONS
```

---

## Features

### Auto-detects your stack

No configuration needed. The command scans your project for `CLAUDE.md`, `package.json`, `requirements.txt`, `go.mod`, `composer.json`, `Cargo.toml`, `docker-compose.yml`, monorepo indicators, and more — then passes that context to all three precogs.

Works with any stack: React, Vue, Angular, FastAPI, Django, Rails, Laravel, .NET, Go, Rust, monorepos, multi-repos.

### Codebase-aware, not just diff-aware

Unlike simple diff reviewers, the precogs have full access to your codebase via tools. They can:

- **Grep** for existing patterns the diff should follow
- **Read** surrounding files to verify assumptions
- **Check** if an endpoint referenced in an error message actually exists

This is how Arthur found an existing `DegradationFlags` pattern that a new feature bypassed — it wasn't in the diff, but it was in the codebase.

### Confidence scoring

Every finding is rated 1-10:

| Score | Meaning |
|-------|---------|
| 9-10 | Verified by reading the code — confirmed issue |
| 7-8 | High likelihood, didn't verify every assumption |
| 5-6 | Reasonable concern, depends on context |
| 3-4 | Speculative — could be an issue under certain conditions |
| 1-2 | Theoretical — general risk pattern |

Anderton uses confidence scores to filter noise: speculative findings (< 4) are separated from confirmed issues. The recommendation accounts for both severity and confidence.

### Commit context

The command includes `git log` output so precogs understand the *intent* behind changes — not just *what* changed. A commit message like "refactor: extract auth middleware" tells Agatha this isn't supposed to change behavior.

### PR metadata

When reviewing by PR number, the command fetches the PR title, description, labels, and author via `gh pr view` — giving precogs additional context about what the PR is trying to accomplish.

---

## What it catches

Real examples from production use:

- **Score scale mismatch** — backend changed from 0-5 to 0-10 storage, frontend still dividing by 2
- **Existing pattern bypassed** — new code used ad-hoc metadata keys instead of the project's typed `DegradationFlags` model
- **Stale failure flag** — error flag written on failure but never cleared on successful retry
- **Wrong endpoint in error message** — error suggested `POST /sessions/{id}/summary/trigger` but the actual route included `/api/v1/` prefix
- **`finally` block race condition** — overwrote failure metadata written in the `except` block
- **Raw exception leak** — `str(e)` from Azure SDK stored in session and returned verbatim to API consumers

---

## Comparison with other tools

| Feature | Minority Report | Anthropic `/code-review` | BugBot (Cursor) |
|---------|----------------|--------------------------|-----------------|
| Independent reviewers | 3 specialized | 5 specialized | 8 parallel passes |
| Codebase exploration | Yes (grep, read, glob) | Limited | Agentic (v2) |
| Confidence scoring | 1-10 per finding | 0-100 threshold | Majority voting |
| Minority reports | Highlighted | Filtered out | Filtered out |
| Cross-repo checking | Yes | No | No |
| Theming | Precogs | Generic | Generic |
| Cost | ~3 Sonnet calls | ~5 Sonnet calls | Included with Cursor |
| Setup | Copy one file | GitHub App install | GitHub App install |

**Key difference**: BugBot and Anthropic's plugin filter out single-reviewer findings to reduce noise. Minority Report keeps them — because sometimes the dissenting opinion is the one that matters.

---

## Customization

### Add project-specific checks

The precog prompts in `mr.md` are plain text. Add checks relevant to your project:

```markdown
> - All API responses must include `request_id` for tracing
> - Database queries must use the repository pattern, never direct ORM calls
> - Feature flags must be checked via the `FeatureService`, not environment variables
```

### Adjust confidence thresholds

In Step 4 (Anderton's Report), modify the recommendation logic:

```markdown
- Any CRITICAL finding with confidence >= 7 → REQUEST CHANGES
- 2+ HIGH findings with confidence >= 7 → REQUEST CHANGES
```

Lower the threshold for stricter reviews, raise it for more lenient ones.

### Change the model

By default, precogs run on whatever model Claude Code is configured to use (typically Sonnet for subagents). The synthesis runs on your main conversation model.

---

## Requirements

- [Claude Code](https://claude.com/code) (CLI)
- `gh` CLI (optional — only needed for reviewing PRs by number)

---

## License

MIT — use it, modify it, share it.

---

*Inspired by the precogs from Minority Report (2002). In the movie, the minority report was the dissenting vision that proved the majority wrong. In code review, it's the finding that only one reviewer caught — and it might be the most important one.*

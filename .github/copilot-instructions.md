# GitHub Copilot Code Review Instructions

## Review Philosophy

- Only comment when you have HIGH CONFIDENCE (>80%) that an issue exists
- Be concise: one sentence per comment when possible
- Focus on actionable feedback, not observations
- When reviewing text, only comment on clarity issues if the text is genuinely confusing or could lead to errors.
  "Could be clearer" is not the same as "is confusing" — stay silent unless HIGH confidence it will cause problems

## Priority Areas (Review These)

### Security & Safety

- Unsafe code blocks without justification
- Command injection risks (shell commands, user input)
- Path traversal vulnerabilities
- Credential exposure or hardcoded secrets
- Missing input validation on external / agent-supplied data
- Improper error handling that could leak sensitive info
- Mutating MCP tools that skip or weaken the `confirmed=False` preview gate
- Persona / policy changes that broaden write access unexpectedly

### Correctness Issues

- Logic errors that could cause incorrect tool results or cluster mutations
- Resource leaks (files, connections, temporary directories)
- Off-by-one errors or boundary conditions
- Optional types that do not need to be optional
- Overly defensive code that adds unnecessary checks
- Unnecessary comments that just restate what the code already shows (remove them)
- Broken SDK contract assumptions (Trainer client / CRD interactions)

### Architecture & Patterns

- Code that violates existing patterns in `kubeflow_mcp/core` or client modules
- New tools not registered in `TOOLS` / descriptions / annotations / personas
- Missing error handling or inconsistent use of `ToolError` / `PreviewResponse`
- Code that is not following [Python PEP8](https://peps.python.org/pep-0008/) guidelines
  (only when it affects readability or correctness beyond what ruff enforces)

## Project-Specific Context

- See [AGENTS.md](../AGENTS.md) in the root directory for project guidelines and architecture decisions.
- Clients are plugins loaded by `core/server.py`; prefer extending a client module over
  putting domain logic into core unless it is truly cross-cutting.

## CI Pipeline Context

**Important**: You review PRs immediately, before CI completes. Do not flag issues that CI will catch.

### What Our CI Checks

- `.github/workflows/test-python.yaml` — pre-commit, `make verify`, unit tests, pip-audit, coverage
- `.github/workflows/check-pr-title.yaml` — conventional PR titles (`chore` / `fix` / `feat` / `revert`)

## Skip These (IMPORTANT)

Do not comment on:

- **Style/formatting** — CI handles this (ruff / pre-commit)
- **Test failures** — CI handles this (pytest matrix)
- **Missing dependencies** — CI / lockfile checks handle this
- **PR title format** — dedicated title workflow handles this

## Response Format

When you identify an issue:

1. **State the problem** (1 sentence)
2. **Why it matters** (1 sentence, only if not obvious)
3. **Suggested fix** (code snippet or specific action)

## When to Stay Silent

If you're uncertain whether something is an issue, don't comment. False positives create noise and reduce trust in the review process.

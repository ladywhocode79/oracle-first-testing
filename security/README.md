# Security

MCP (Model Context Protocol) threat model and least-privilege patterns.

## Why This Matters

When an AI agent runs tests, it needs access to:
- Browser automation (Playwright, Selenium)
- APIs (staging endpoints, test data services)
- Filesystems (test code, results, logs)
- Credentials (API keys, auth tokens)

This access can be weaponized if not scoped correctly. This directory documents how to grant agents the minimum access they need, and how to detect misuse.

## What's Here

- **mcp-security-guide.md** — Threat model, scoping rules, patterns for safe MCP setup
  - What can an agent do with browser access? (read secrets from the page, delete data, etc.)
  - How do you prevent it? (MCP allowlists, read-only vs. mutating operations)
  - When to use service accounts vs. user credentials
  - How to audit what an agent actually did

## Principles

1. **Least privilege** — Every MCP server gets only the access it needs
2. **Audit trail** — Every action the agent takes is logged
3. **Fail closed** — If an action is not explicitly allowed, it fails
4. **Credentials are data** — Test data, API keys, session tokens are treated as untrusted input

---

*Last updated: Phase 0 — skeleton created*


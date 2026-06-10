# Concepts

Architectural doctrine and durable knowledge for AI-driven test automation.

These are tool-agnostic principles and patterns that apply whether you're using Claude, another LLM, or building hybrid human-AI workflows. They are the foundation that all skills and prompts are built on.

## The Docs Here

- **oracle-executor-principle.md** — The core design principle. Why we split oracle (the human/reasoning checklist) from executor (the agent). Research-backed, essential reading.
- **five-lifecycle-stages.md** — Plan → Author → Execute → Heal → Analyze. How AI fits into each stage.
- **agentic-vs-scaffolding.md** — When to use a free-roaming agent vs. step-by-step scaffolding. Tradeoffs and when each wins.
- **mcp-for-testing.md** — How Model Context Protocol enables safe, scoped agent access to test runners, APIs, and browsers.
- **self-healing-strategy.md** — Diagnosis-first repair. How to debug and fix flaky tests with AI.

---

**How to use this directory:**

- Reading docs here prepares you to understand `/prompts` (the specific prompt patterns).
- Before choosing a skill, read the concept that backs it.
- When contributing a new prompt pattern, add the doctrine first.


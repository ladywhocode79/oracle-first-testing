# Skills

Claude Code Skills — installable, runnable automation for testing.

## What Are Skills?

Skills are Claude Code extensions that encapsulate a full workflow. They consume prompts from `/prompts` and doctrine from `/concepts`, and provide a single entry point for users.

For example:
- **automation-architect** — Scaffold a full testing framework from scratch (Python, Java, or mocked services)
- **automation-architect-python** — Python/Pytest-specific scaffolding
- **automation-architect-java** — Java/TestNG-specific scaffolding
- **automation-architect-mock** — WireMock + mock service setup
- **automation-architect-nfr** — Load testing, performance, chaos, security testing

## How Skills Relate to This Repo

```
Concepts (doctrine)
    ↓
Prompts (pattern library)
    ↓
Skills (installable, runnable)
```

- **Skills reference, don't duplicate.** A skill's `SKILL.md` points to the prompts it uses, not copying them.
- **Skills are consumers.** They adapt prompts for a specific context (e.g., "Playwright for Python" or "WireMock setup").
- **Prompts are the source of truth.** If a prompt changes, all skills that reference it automatically benefit.

## Phase 0 Work

These skills are being **refactored** (not replaced) to align with the new prompt library structure. Progress:

- [ ] `automation-architect` — Refactor to reference `/prompts`
- [ ] `automation-architect-python` — Refactor to reference `/prompts`
- [ ] `automation-architect-java` — Refactor to reference `/prompts`
- [ ] `automation-architect-mock` — Refactor to reference `/prompts`
- [ ] `automation-architect-nfr` — Refactor to reference `/prompts`

## Installing a Skill

See the main [README.md](../README.md) for installation instructions.

---

*Last updated: Phase 0 — skeleton created, refactoring planned*


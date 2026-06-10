# NORTH-STAR.md
> The guiding document for `create-test-automation-skills` — what it is, why it exists, where it's going, and the principles every contributor navigates by.

---

## What This Repo Is

A **master reference for prompt-based and agentic test automation** — built for any tester, at any level, to learn, adapt, and apply.

It ships working **Claude Code Skills** (installable, runnable), but the skills are one pillar, not the whole building. The deeper value is the knowledge layer underneath them: prompt patterns, architectural doctrine, lifecycle guides, and worked examples that any tester can read, learn from, and apply regardless of which AI tool they use.

**The short version:** come here to understand how to test with AI. Leave with either knowledge, working code, or both.

---

## Who This Is For

| Persona | What they get here |
|---|---|
| Tester new to AI | Concepts, glossary, decision trees, learning path |
| SDET / automation engineer | Prompt library, skills, architecture patterns |
| QA lead / architect | Doctrine docs, tradeoff guides, CI/CD patterns |
| Contributor | Clear structure, `CONTRIBUTING.md`, phase roadmap |

---

## The Core Design Principle (Read This First)

> **Separate the oracle from the executor.**

Research shows AI agents score under 30% F1 detecting defects when left to plan and execute alone. When given a human-authored checklist, the same model nearly doubles its score.

The reason: LLMs are biased toward predicting "no defect" — because training data favours success states. When the model both writes the test plan *and* judges the result, that bias compounds. It misses what it didn't think to look for.

**The rule this gives us, applied everywhere in this repo:**

- **Humans (or a dedicated reasoning pass) own the oracle** — *what* to verify, acceptance criteria, risk areas, edge cases.
- **The agent owns execution and reporting** — *running* against the oracle, surfacing deviations, not inventing scope.

Every skill, every prompt, every guide in this repo is built around this split. It is not a workaround for AI's limitations — it is the correct architecture for reliable automated testing, human or AI.

---

## Repo Structure

```
/
├── NORTH-STAR.md          ← you are here
├── README.md              ← entry point for any visitor
├── CONTRIBUTING.md        ← how to add to this repo
│
├── concepts/              ← durable knowledge, tool-agnostic
│   ├── oracle-executor-principle.md
│   ├── five-lifecycle-stages.md
│   ├── agentic-vs-scaffolding.md
│   ├── mcp-for-testing.md
│   └── self-healing-strategy.md
│
├── prompts/               ← versioned prompt library (the heart)
│   ├── plan/              ← requirement → oracle, risk analysis, test scope
│   ├── author/            ← test generation, data factories, contract tests
│   ├── execute/           ← MCP/CLI execution patterns, agentic runs
│   ├── heal/              ← self-healing, diagnosis-first repair
│   └── analyze/           ← failure triage, flaky detection, reporting
│
├── skills/                ← Claude Code Skills (consumers of /prompts)
│   ├── automation-architect/
│   ├── automation-architect-python/
│   ├── automation-architect-java/
│   ├── automation-architect-mock/
│   └── automation-architect-nfr/
│
├── guides/                ← learning path, decision trees, glossary
│   ├── learning-path.md
│   ├── decision-tree-what-to-automate.md
│   └── glossary.md
│
├── examples/              ← end-to-end worked runs
│   └── sample-prd/        ← (existing) PRD → oracle → execution → report
│
└── security/              ← MCP threat model, least-privilege patterns
    └── mcp-security-guide.md
```

---

## What Changed From Phase 1

Previously this repo **was** the Claude Code Skills suite. The skills contained the prompt logic, the architecture decisions, and the knowledge all bundled together inside `SKILL.md` files.

**The shift:** `/skills` is now a *consumer* of `/prompts` and `/concepts`. Prompt logic lives in the prompt library. Architectural doctrine lives in `/concepts`. The skills reference both — so a tester reading a prompt and the skill executing it are using the same artifact. No drift. Single source of truth.

The existing skills are **preserved and will be refactored** to align with this structure over the phases below. Nothing is thrown away.

---

## Phase Roadmap

| Phase | Name | Deliverable | Status |
|---|---|---|---|
| 0 | Reframe & Restructure | Directory skeleton, new README, this doc, oracle-executor doctrine | 🔄 In Progress |
| 1 | Prompt Library | `/prompts` tree — versioned, annotated, one prompt per lifecycle stage to start | ⬜ Planned |
| 2 | MCP Execution Layer | Playwright MCP + CLI track, token tradeoff doc, worked example vs sample-prd | ⬜ Planned |
| 3 | Oracle-First Workflow | Requirement → checklist → agent execution pipeline, wired to sample-prd | ⬜ Planned |
| 4 | Self-Healing + Analysis | Diagnosis-first healing reference, Allure + JUnit XML reporting layer | ⬜ Planned |
| 5 | Modernise Tracks | Playwright for Java, Appium, contract testing, synthetic data, visual validation | ⬜ Planned |
| 6 | NFR + Security | Flesh out gated NFR skill, ship MCP security guide | ⬜ Planned |
| 7 | Make It Teachable | Learning path, decision trees, glossary, CONTRIBUTING.md, "start here" guide | ⬜ Planned |

---

## Pairing Model

This repo is built in **anchor / shadow** mode:

- **Anchor (Claude):** proposes architecture, writes first drafts, explains every decision and its tradeoffs.
- **Shadow (contributor):** reviews, questions, implements refinements, owns the `git push`.

The goal is not just a good repo — it is a tester who understands *why* every decision was made. When in doubt, ask "why" before merging.

---

## Guiding Principles

1. **Oracle first, executor second.** Always. See above.
2. **Knowledge is the product; skills are a delivery mechanism.** The prompt library and docs outlive any one AI tool.
3. **Honest over hyped.** Every prompt and guide acknowledges where AI helps and where it doesn't. We cite research. We show failure modes alongside success modes.
4. **Single source of truth.** Prompt logic lives in `/prompts`. Doctrine lives in `/concepts`. Skills reference, never duplicate.
5. **Any tester, not just Claude Code users.** Structure, language, and examples must be approachable without assuming familiarity with any specific AI tool.
6. **Least privilege everywhere.** MCP servers get only the access they need. Test data is treated as untrusted input.

---

## How to Navigate This Repo

- **New to AI testing?** → Start at `guides/learning-path.md`
- **Want to run something now?** → Start at `README.md` → install a skill
- **Want to understand the architecture?** → Start at `concepts/`
- **Looking for a prompt?** → Go to `prompts/` → pick your lifecycle stage
- **Want to contribute?** → Read `CONTRIBUTING.md` first

---

*Last updated: Phase 0 — initial restructure.*
*Maintained by: ladywhocode79 + Claude (anchor/shadow pair).*

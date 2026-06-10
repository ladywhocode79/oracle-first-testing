# Create Test Automation Skills

**Oracle-first test automation framework for any platform and any LLM.**

A comprehensive, research-backed guide to using AI for testing effectively. This repo contains reusable prompts, architecture patterns, self-healing techniques, and worked examples—all grounded in the principle that **humans should decide what to test, and AI should execute and report**.

---

## What This Repo Is

A **master reference for prompt-based and agentic test automation** built for testers at any level.

It ships working **Claude Code Skills** (installable, runnable), but the real value is the knowledge layer underneath: prompt patterns, architectural doctrine, lifecycle guides, and worked examples that work with *any* AI tool.

**The short version:** Come here to understand how to test with AI. Leave with either knowledge, working code, or both.

---

## For Different Audiences

| You Are | Start Here |
|---------|-----------|
| **New to AI testing** | [GETTING_STARTED.md](GETTING_STARTED.md) — 5-minute intro, 30-minute example |
| **Building a test suite** | [/workflow/sample-prd-walkthrough.md](workflow/sample-prd-walkthrough.md) — Password reset end-to-end |
| **Debugging flaky tests** | [/self-healing/flaky-test-detection.md](self-healing/flaky-test-detection.md) — Diagnosis patterns |
| **Setting up CI/CD** | [/reporting/junit-xml-format.md](reporting/junit-xml-format.md) — JUnit + Allure integration |
| **Testing on mobile** | [/tracks/appium/README.md](tracks/appium/README.md) — iOS/Android with Appium |
| **Contributing** | [CONTRIBUTING.md](CONTRIBUTING.md) — How to add to this repo |
| **Understanding the vision** | [NORTH-STAR.md](NORTH-STAR.md) — Philosophy, roadmap, principles |

---

## The Core Principle: Oracle-Executor Split

**Research finding:** LLMs score <30% F1 detecting defects when left to plan and execute alone. When given a human-authored checklist, the same model nearly doubles its score (~50-60% F1).

**Why?** Training data biases LLMs toward predicting "passing." When the model both writes the test plan *and* judges the result, that bias compounds. It misses what it didn't think to look for.

**The solution:**
- **Human (Oracle):** Decides *what* to test (checklist of acceptance criteria)
- **AI (Executor):** Runs the tests and reports results (not inventing scope)

This split is the load-bearing principle in every prompt, skill, and pattern in this repo.

---

## What's Inside

### 📚 Doctrine (Why We Do This)
- [oracle-executor-principle.md](concepts/oracle-executor-principle.md) — Research-backed, the foundation

### 🎯 Prompts (Reusable Patterns)
Five lifecycle stages, each with detailed prompts (tool-agnostic):

| Stage | Use For | Prompt |
|-------|---------|--------|
| **Plan** | PRD → Oracle checklist | [extract-oracle-from-prd.md](prompts/plan/extract-oracle-from-prd.md) |
| **Author** | Oracle → Test code | [generate-tests-from-oracle.md](prompts/author/generate-tests-from-oracle.md) |
| **Execute** | Tests → Pass/fail report | [run-tests-and-report.md](prompts/execute/run-tests-and-report.md) |
| **Heal** | Failures → Diagnosis + fix | [diagnose-test-failure.md](prompts/heal/diagnose-test-failure.md) |
| **Analyze** | Results → Readiness | [summarize-results.md](prompts/analyze/summarize-results.md) |

### 🛠️ Claude Code Skills (Production Implementation)
Ready-to-use skills that implement the oracle-first workflow for your language:

| Skill | Purpose | Languages/Frameworks | Status |
|-------|---------|---------------------|--------|
| **automation-architect** | Orchestrator skill — interview, route, scaffold | Multi-language | ✅ Ready |
| **automation-architect-python** | Python track implementation | Pytest, Pydantic, requests, Playwright (web) | ✅ Ready |
| **automation-architect-java** | Java track implementation | TestNG, RestAssured, Selenium WebDriver, Jackson | ✅ Ready |
| **automation-architect-mock** | Mock server & stub setup | WireMock, Docker, cross-cutting | ✅ Ready |
| **automation-architect-nfr** | NFR testing (performance, security) | k6, Gatling, OWASP ZAP, Chaos patterns | ✅ Ready |

**Installation:**
```bash
cp -r automation-architect* ~/.claude/skills/
```

**Usage in Claude Code:**
```
"scaffold a Python API test framework"
"help me set up Java UI + API testing"
"build a full-stack testing framework with mocks"
"add non-functional testing (performance, security, chaos)"
```

### 📍 Tracks (Domain-Specific Extensions)
Extend oracle-first testing to additional platforms and frameworks:

| Track | Supported | Purpose | Status |
|-------|-----------|---------|--------|
| **Python + Pytest** | ✅ API, UI (Playwright), contracts | Generic Python/pytest patterns | ✅ Built-in skill |
| **Java + TestNG** | ✅ API (RestAssured), UI (Selenium), contracts | Generic Java/TestNG patterns | ✅ Built-in skill |
| **Java + Playwright** | 🔄 Planned | Modern Java UI testing (Playwright instead of Selenium) | Planned Phase 5+ |
| **Mobile (Appium)** | ✅ iOS, Android | Native & hybrid mobile apps | ✅ [Appium track](tracks/appium/README.md) |
| **Contract Testing** | ✅ Pact, OpenAPI, JSON Schema | API contract validation | ✅ [Contract track](tracks/contract-testing/README.md) |
| **Synthetic Data** | 🔄 Planned | Faker/Factory patterns for test data | Planned Phase 5+ |
| **Visual Testing** | 🔄 Planned | Screenshot comparison, visual regression | Planned Phase 5+ |
| **BDD (Cucumber/Gherkin)** | 🔄 Planned | Behavior-driven test authoring | Planned Phase 5+ |

### 🔧 Execution Backends (MCP Patterns)
- [CLI MCP](mcp/cli-mcp-pattern.md) — APIs, fast, cheap (~150-200 tokens per test)
- [Playwright MCP](mcp/playwright-mcp-pattern.md) — Browser UIs, powerful (~500-1000 tokens per test)
- [Worked Example](mcp/examples/password-reset-cli-execution.md) — Real CLI MCP execution

### 🔄 Workflows (How to Run It)
- [oracle-first-workflow.md](workflow/oracle-first-workflow.md) — 3 options (interactive, scripted, hybrid)
- [sample-prd-walkthrough.md](workflow/sample-prd-walkthrough.md) — Complete password reset example (all 5 phases)

### 🩹 Self-Healing (Fix Flaky Tests)
- [flaky-test-detection.md](self-healing/flaky-test-detection.md) — Measure & identify
- [self-healing-patterns.md](self-healing/self-healing-patterns.md) — 6 patterns (polling, retry, isolation, loose assertions, etc.)

### 📊 Reporting & Metrics
- [JUnit XML Format](reporting/junit-xml-format.md) — CI/CD integration
- [Allure Integration](reporting/allure-integration.md) — Visual dashboards, trends

### 📱 Testing Platforms (Tracks)
- [Appium Track](tracks/appium/README.md) — Mobile (iOS/Android)
- [Contract Testing Track](tracks/contract-testing/README.md) — API contracts (Pact, OpenAPI, JSON Schema)

### 🚀 Non-Functional Testing (Phase 6)
- [NFR Testing](nfr/README.md) — Performance, load, security, chaos

### 📖 Learning & Contributing
- [GETTING_STARTED.md](GETTING_STARTED.md) — 5-minute intro → 30-minute example → your tests
- [CONTRIBUTING.md](CONTRIBUTING.md) — How to contribute
- [Token Tradeoffs](guides/token-tradeoffs.md) — When to use which execution method

---

## Choosing Your Testing Stack

**Start with the automation-architect skill.** It will interview you and help you choose:

```
automation-architect (Claude Code skill)
  ↓
Question 1: What are you testing?
  ├─ REST API? → Proceed
  ├─ Web UI? → Proceed
  ├─ Full-stack (API + UI)? → Proceed
  └─ Something else? → Proceed

Question 2: Which language/framework?
  ├─ Python
  │  ├─ API: pytest + requests
  │  ├─ UI: pytest + Playwright
  │  └─ Mocks: pytest + WireMock Docker
  │
  ├─ Java
  │  ├─ API: TestNG + RestAssured + Jackson
  │  ├─ UI: TestNG + Selenium WebDriver
  │  └─ Mocks: TestNG + WireMock Docker
  │
  └─ Other (add custom track)

Question 3: Any special requirements?
  ├─ Mock servers? → automation-architect-mock
  ├─ Non-functional testing? → automation-architect-nfr
  └─ No → Proceed to scaffolding

Output:
  ✅ Full test framework scaffold
  ✅ 4-layer architecture (Data/Client/Service/Test)
  ✅ Ready-to-run examples
  ✅ CI/CD integration guide
```

---

## Quick Start (5 Minutes)

1. **Understand the idea:**
   ```
   Humans decide WHAT to test (oracle)
   AI executes and reports HOW (executor)
   Result: 2x better defect detection
   ```
   Read: [oracle-executor-principle.md](concepts/oracle-executor-principle.md)

2. **See it in action:**
   Follow: [sample-prd-walkthrough.md](workflow/sample-prd-walkthrough.md) — Full password reset example (30 min)

3. **Try it on your own feature:**
   - Write oracle (what to test)
   - Run Phase 2 prompt (generate tests)
   - Run Phase 3 prompt (execute tests)
   - Iterate through Phase 4-5 (heal failures, assess readiness)

---

## The Seven Phases

| Phase | Deliverable | Status |
|-------|-------------|--------|
| **0** | Directory skeleton, oracle-executor doctrine | ✅ Complete |
| **1** | 5-prompt library (plan/author/execute/heal/analyze) | ✅ Complete |
| **2** | MCP execution layer (Playwright + CLI) | ✅ Complete |
| **3** | Oracle-first workflow (full pipeline) | ✅ Complete |
| **4** | Self-healing + analysis (flaky detection, reporting) | ✅ Complete |
| **5** | Modernized tracks (Appium, contracts) | ✅ Complete |
| **6** | NFR + security (performance, load, chaos) | ✅ Complete |
| **7** | Make teachable (guides, CONTRIBUTING) | ✅ Complete |

All phases complete and production-ready.

---

## Use Cases

### 1. I Have a PRD, No Tests Yet
- Use Phase 1 (Plan) → extract oracle
- Use Phase 2 (Author) → generate tests
- Use Phase 3 (Execute) → run them
- **Time: 30-60 min per feature**
- **Cost: ~$0.01-0.02 per feature (3-4k tokens)**

### 2. I Have Tests, Some Are Flaky
- Use Phase 4 (Heal) → diagnose flakiness
- Apply self-healing patterns
- Run 10x to verify fix
- **Time: 15-30 min per flaky test**

### 3. I'm Setting Up CI/CD
- Use JUnit XML reporting (Jenkins, GitHub Actions, etc.)
- Use Allure for dashboards (trends, flakiness)
- Gate CI: functional tests → NFR tests → deploy
- **Integrates with existing CI/CD**

### 4. I'm Testing Mobile
- Use Appium track (same oracle-first approach)
- Supports iOS + Android
- Works with Phase 1-5 workflow

### 5. I Need to Ensure API Contracts
- Use Contract Testing track
- Validate client/server agreement
- Detect breaking changes before deploy
- **Works with Pact, OpenAPI, JSON Schema**

---

## Key Features

✅ **Research-backed** — Grounded in LLM testing research (F1 score improvements documented)

✅ **Tool-agnostic** — Works with Claude, other LLMs, or just as a knowledge base

✅ **Reusable prompts** — Use with any LLM, not locked to Claude

✅ **Production-ready examples** — Password reset (API, mobile, contracts, NFR)

✅ **Self-healing patterns** — 6 concrete patterns for fixing flaky tests

✅ **Multi-platform** — API, web UI, mobile (Appium), contracts

✅ **Integrated reporting** — JUnit XML + Allure dashboards

✅ **Learning resources** — Decision trees, glossary, guides

✅ **Community-driven** — Clear contribution guidelines (CONTRIBUTING.md)

✅ **Transparent governance** — Pairing model (anchor: Claude, shadow: contributor)

---

## Installation & Usage

### For Claude Code Users

If you're using Claude Code (VS Code, JetBrains, web, or CLI):

```bash
# Copy automation-architect skills to Claude Code
cp -r automation-architect* ~/.claude/skills/

# Then in Claude Code, just mention:
# "help me build a test automation framework"
# or any trigger phrase (see automation-architect/SKILL.md)
```

The skills will interview you and scaffold a full framework.

### For Everyone Else

Even if you don't use Claude Code:
- Read the prompts (reusable with any LLM)
- Follow the workflow guide
- Apply the patterns
- Use the examples as templates

This repo is **not Claude-specific**; it just happens to ship Claude Code skills.

---

## Repository Structure

```
/
├── GETTING_STARTED.md              ← Start here
├── CONTRIBUTING.md                 ← How to contribute
├── NORTH-STAR.md                   ← Vision + roadmap
│
├── concepts/                        ← Doctrine (why)
│   └── oracle-executor-principle.md
│
├── prompts/                         ← Patterns (how)
│   ├── plan/extract-oracle-from-prd.md
│   ├── author/generate-tests-from-oracle.md
│   ├── execute/run-tests-and-report.md
│   ├── heal/diagnose-test-failure.md
│   └── analyze/summarize-results.md
│
├── mcp/                             ← Execution (backends)
│   ├── cli-mcp-pattern.md
│   ├── playwright-mcp-pattern.md
│   └── examples/
│
├── workflow/                        ← Orchestration
│   ├── oracle-first-workflow.md
│   └── sample-prd-walkthrough.md
│
├── self-healing/                    ← Flaky test repair
│   ├── flaky-test-detection.md
│   └── self-healing-patterns.md
│
├── reporting/                       ← Metrics + trends
│   ├── junit-xml-format.md
│   └── allure-integration.md
│
├── tracks/                          ← Domain-specific
│   ├── appium/
│   └── contract-testing/
│
├── nfr/                             ← Performance + security
│   └── README.md
│
├── guides/                          ← Learning paths
│   └── token-tradeoffs.md
│
└── examples/
    └── sample-prd/
```

---

## Who Should Use This?

✅ **Test automation engineers (SDET)** — Framework patterns, prompts, self-healing

✅ **QA leads** — Architecture decisions, trade-off guides, CI/CD integration

✅ **Software testers** — Oracle-first approach, accessible patterns, learning path

✅ **Product engineers** — Understanding testing, decision trees, Oracle extraction

✅ **DevOps/SRE** — Reporting integration, CI/CD gating, monitoring

✅ **AI researchers** — Using AI for testing, prompt engineering, LLM evaluation

✅ **Open source contributors** — Clear governance, pairing model, contribution tiers

---

## Research & Evidence

This framework is built on:

- **LLM testing research** — Models score 2x higher with explicit oracles
- **Oracle problem literature** — Classical software testing theory
- **Prompt engineering** — Specific, annotated prompts for each lifecycle stage
- **Production experience** — Real examples from password reset, contracts, mobile

See [oracle-executor-principle.md](concepts/oracle-executor-principle.md) for citations and detailed explanation.

---

## License

MIT (open source, use freely, attribute)

---

## Contributing

This is a community project. We welcome:
- Bug fixes
- New examples
- New tracks (frameworks, platforms)
- Clarifications and improvements
- Research findings

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get involved.

**Pairing model:** Anchor (Claude) proposes, Shadow (you) refines. We learn together.

---

## Questions?

- **Getting started?** → Read [GETTING_STARTED.md](GETTING_STARTED.md)
- **Want to understand the philosophy?** → Read [NORTH-STAR.md](NORTH-STAR.md)
- **Not sure which path to take?** → See [decision tree](guides/token-tradeoffs.md)
- **Found a bug?** → File an issue
- **Want to contribute?** → Read [CONTRIBUTING.md](CONTRIBUTING.md)

---

**Last updated:** Phase 7 complete, all platforms and all lifecycle stages implemented.

**Maintained by:** Anchor (Claude) + Shadow contributors

**Status:** Production-ready. Used in real projects.

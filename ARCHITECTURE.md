# Architecture: Three Usage Modes

This framework can be used **three distinct ways**. Understand which one fits your needs.

---

## Three Modes of Usage

```
MODE 1: SKILLS ONLY                MODE 2: PROMPTS ONLY           MODE 3: BOTH
(Claude Code Users)                (Any LLM Users)                (Claude Code + Control)

User: "Scaffold Python tests"       User: reads /prompts/          User: "Scaffold tests"
         ↓                                 ↓                              ↓
Skill interviews                    Step 1: /prompts/plan/         Skill scaffolds
  ↓                                 Step 2: /prompts/author/       Then uses /prompts/
Skill chains prompts                Step 3: /prompts/execute/      for advanced patterns
  ↓                                 Step 4: /prompts/heal/
Skill adds Python scaffold          Step 5: /prompts/analyze/      Result:
  ↓                                 ↓                              Framework + Flexibility
Output: Complete framework          Output: Working framework

FAST (2 min)                        FLEXIBLE (10 min)              BALANCED (5 min)
LOW CONTROL                         HIGH CONTROL                   HIGH CONTROL
```

---

## How the Framework Is Organized

How the oracle-first framework is organized and how the pieces fit together.

---

## Three Layers

```
┌─────────────────────────────────────────────────────────────┐
│ LAYER 1: DOCTRINE                                           │
│ (Why we do this)                                            │
│ - oracle-executor-principle.md                              │
│ - Research: LLM effectiveness with explicit oracles         │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 2: PROMPTS                                            │
│ (Generic patterns, tool-agnostic)                           │
│ - /prompts/plan/              (extract oracle)              │
│ - /prompts/author/            (generate tests)              │
│ - /prompts/execute/           (run tests)                   │
│ - /prompts/heal/              (diagnose failures)           │
│ - /prompts/analyze/           (assess readiness)            │
│ → Works with ANY LLM (Claude, GPT, others)                 │
│ → Works with ANY language (Python, Java, JS, etc.)         │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 3A: SKILLS (Claude Code)                              │
│ (Implementation for Claude Code users)                       │
│ - automation-architect         (orchestrator)                │
│ - automation-architect-python  (Python/pytest track)        │
│ - automation-architect-java    (Java/TestNG track)          │
│ - automation-architect-mock    (WireMock setup)             │
│ - automation-architect-nfr     (Performance, security)      │
│ → Encapsulates prompts + language-specific logic            │
│ → Scaffolds full frameworks                                 │
└─────────────────────────────────────────────────────────────┘
                             ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 3B: TRACKS (Extensible)                               │
│ (Domain-specific patterns)                                   │
│ - /tracks/appium/              (mobile: iOS/Android)        │
│ - /tracks/contract-testing/    (API contracts)              │
│ - /tracks/playwright-java/     (Java UI, modern)            │
│ - /tracks/bdd/                 (Cucumber/Gherkin)           │
│ - [more to be added]                                        │
│ → Works with any LLM + any language                         │
│ → Extends the prompts to new domains                        │
└─────────────────────────────────────────────────────────────┘
```

---

## How They Connect

### Example: Testing a Password Reset API

**Path A: Using Claude Code Skill (Easy)**
```
User: "help me scaffold a Python API test framework"
       ↓
automation-architect (Claude Code skill)
       ↓
Prompts from /prompts/ (embedded in skill)
       ↓
automation-architect-python (language track)
       ↓
Output: Full pytest + requests framework scaffold
```

**Path B: Using Generic Prompts (Flexible)**
```
User: "I want to test password reset"
       ↓
Human reads: /prompts/plan/extract-oracle-from-prd.md
       ↓
Extracts oracle (checklist)
       ↓
Claude (or any LLM) reads: /prompts/author/generate-tests-from-oracle.md
       ↓
Generates test code (any language)
       ↓
Runs tests, collects results
       ↓
Uses: /prompts/heal/ and /prompts/analyze/
       ↓
Output: Working test suite + readiness assessment
```

**Path C: Using a Track (Domain-Specific)**
```
User: "I want to test a mobile app"
       ↓
Reads: /tracks/appium/README.md
       ↓
Learns mobile-specific oracle claims
       ↓
Uses: /prompts/ (adapted for mobile)
       ↓
Generates Appium test code
       ↓
Output: iOS/Android test suite
```

---

## Skills Architecture (Detailed)

Each skill is a wrapper around the oracle-first prompts:

```
automation-architect/SKILL.md
  ├─ Frontmatter: skill metadata
  ├─ Discovery interview (what are you testing?)
  ├─ Routes to appropriate track:
  │  ├─ Python? → automation-architect-python
  │  ├─ Java? → automation-architect-java
  │  └─ Mocks? → automation-architect-mock
  │
  ├─ Uses: /prompts/ (embedded)
  ├─ Adds: language-specific logic
  └─ Outputs: full scaffold

automation-architect-python/SKILL.md
  ├─ Interview: API? UI? Full-stack?
  ├─ Interview: Pytest conventions?
  ├─ Embeds: /prompts/ for Python/pytest
  ├─ Adds:
  │  ├─ 4-layer architecture (Data/Client/Service/Test)
  │  ├─ Pydantic validators
  │  ├─ requests + Playwright patterns
  │  └─ pytest fixtures, markers, plugins
  └─ Outputs: ready-to-run Python test framework

automation-architect-java/SKILL.md
  ├─ Interview: API? UI? Full-stack?
  ├─ Interview: TestNG, Maven/Gradle conventions?
  ├─ Embeds: /prompts/ for Java/TestNG
  ├─ Adds:
  │  ├─ 4-layer architecture
  │  ├─ Jackson JSON handling
  │  ├─ RestAssured patterns
  │  ├─ Selenium WebDriver patterns
  │  └─ TestNG annotations, listeners, reports
  └─ Outputs: ready-to-run Java test framework

automation-architect-mock/SKILL.md
  ├─ Sets up: WireMock in Docker
  ├─ Creates: stub definitions
  ├─ Implements: Strategy pattern (real vs. mock switching)
  ├─ Embeds: /prompts/ for contract testing
  └─ Outputs: mock server + test integration

automation-architect-nfr/SKILL.md
  ├─ Gate: Functional tests must pass first
  ├─ Embeds: /prompts/analyze/ (performance + security)
  ├─ Adds:
  │  ├─ k6/Locust load test patterns
  │  ├─ OWASP ZAP security patterns
  │  ├─ Chaos testing (Toxiproxy)
  │  └─ SLA + percentile assertions
  └─ Outputs: NFR test suite + performance baseline
```

---

## Tracks: Language + Platform Coverage

### Current Tracks

| Track | Language | Framework | Test Type | Status |
|-------|----------|-----------|-----------|--------|
| Python | Python | pytest | API, UI, Full-stack | ✅ Skill-based |
| Java | Java | TestNG | API, UI, Full-stack | ✅ Skill-based |
| Appium | Python/Java | pytest/TestNG + Appium | Mobile | ✅ Doc-based |
| Contract | Python/Java | pytest/TestNG + Pact/OpenAPI | API contracts | ✅ Doc-based |

### Planned Tracks

| Track | Language | Framework | Test Type | Status |
|-------|----------|-----------|-----------|--------|
| Playwright/Java | Java | Playwright + TestNG | UI (modern) | 🔄 Planned |
| BDD | Python/Java | pytest/TestNG + Cucumber | Behavior-driven | 🔄 Planned |
| Synthetic Data | Python/Java | Faker + Factory | Data generation | 🔄 Planned |
| Visual Testing | Python/Java | Applitools/Percy | Visual regression | 🔄 Planned |

---

## How to Use This

### I'm Using Claude Code

1. **Install the skills**
   ```bash
   cp -r automation-architect* ~/.claude/skills/
   ```

2. **Choose your stack via skill interview**
   ```
   "help me set up a Java API testing framework"
   # → automation-architect interviews you
   # → routes to automation-architect-java
   # → scaffolds TestNG + RestAssured framework
   ```

3. **Skill handles the rest**
   - Embeds oracle-first prompts
   - Language-specific scaffolding
   - 4-layer architecture
   - CI/CD setup guide

### I'm Not Using Claude Code

1. **Use the prompts directly**
   - Read `/prompts/plan/` → extract oracle
   - Give oracle to ANY LLM → `/prompts/author/`
   - Run tests, collect results
   - Use `/prompts/heal/` and `/prompts/analyze/`

2. **Extend with tracks as needed**
   - Mobile? → `/tracks/appium/`
   - Contracts? → `/tracks/contract-testing/`
   - Other? → Create your own track

### I Want to Add a New Track

1. **Create directory**
   ```
   /tracks/[track-name]/
   ```

2. **Add documentation**
   - `README.md` — Overview
   - `oracle-adaptation.md` — Domain-specific oracle claims
   - `/examples/` — Worked examples

3. **Reference prompts**
   - Link to `/prompts/` (don't copy)
   - Adapt for your domain (e.g., mobile-specific assertions)

4. **Submit PR** → We review and integrate

---

## The Prompts Are the Core

Notice:
- **Prompts don't change** (generic, reusable)
- **Skills wrap them** (language-specific scaffolding)
- **Tracks extend them** (domain-specific patterns)

This architecture means:
- ✅ Single source of truth (prompts)
- ✅ No drift between skill and doc
- ✅ Easy to add new skills/tracks
- ✅ Works with any LLM or tool

---

## Decision Tree: Which Path?

```
START: What's your situation?

Using Claude Code?
  ├─ YES → Use automation-architect skill
  │        (easiest, most scaffolding)
  │
  └─ NO → Use prompts directly
           (more flexible, works with any LLM)

Want a specific language?
  ├─ Python → Python skill or /prompts/
  ├─ Java → Java skill or /prompts/
  └─ Other → /prompts/ (works with any language)

Want a specific domain?
  ├─ Mobile → /tracks/appium/
  ├─ Contracts → /tracks/contract-testing/
  └─ Other → Create new track or use /prompts/

Want non-functional testing?
  ├─ YES → automation-architect-nfr skill
  └─ NO → Use basic skill/prompts
```

---

## Summary

| What | Purpose | Tool | User |
|-----|---------|------|------|
| **Doctrine** | Why oracle-first works | `/concepts/` | Everyone |
| **Prompts** | Reusable patterns | `/prompts/` | Anyone (any LLM) |
| **Skills** | Claude Code implementation | `automation-architect*` | Claude Code users |
| **Tracks** | Domain-specific patterns | `/tracks/` | Anyone (extensible) |

All four layers work together but can be used independently.

---

## Links

- **Doctrine:** [/concepts/oracle-executor-principle.md](concepts/oracle-executor-principle.md)
- **Prompts:** [/prompts/](prompts/)
- **Skills:** automation-architect* directories
- **Tracks:** [/tracks/](tracks/)

---

*Architecture designed for flexibility: Use what you need, extend what you don't.*


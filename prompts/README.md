# Prompts

Versioned prompt library, organized by lifecycle stage.

This is the actionable heart of the repo. Every prompt here is:
- **Grounded in `/concepts` doctrine** — you'll see references to the oracle-executor principle, the five lifecycle stages, etc.
- **Designed to work standalone** — grab a prompt, adapt it to your context, use it
- **Suitable for any LLM** — Claude, but also others; they're written to be portable
- **Annotated with tradeoffs** — cost, latency, accuracy, what it assumes, what it doesn't handle

## The Five Lifecycle Stages

### `/plan` — Plan & Risk Analysis
From requirement (PRD, user story, API spec) to **oracle**: acceptance criteria, test scope, edge cases, risk areas.

**Prompts here:**
- Summarize a requirement, extract testable claims
- Identify edge cases from domain (data validation, security, concurrency, etc.)
- Generate a risk-ranked checklist
- Create a contract/API spec from documentation

### `/author` — Test Authoring
From oracle (checklist) to **test code**: scaffolding, data factories, parametrization, helper functions.

**Prompts here:**
- Generate test functions from a checklist
- Create synthetic data and factories
- Write contract tests (schema validation)
- Scaffold UI/API test layers

### `/execute` — Test Execution & Reporting
**Agentic running** of tests: MCP-driven execution, log parsing, result collection, failure diagnosis.

**Prompts here:**
- Execute a test suite and parse results
- Drive a browser or API via MCP
- Collect and parse logs in real time
- Build an execution report (JSON, JUnit XML, Allure)

### `/heal` — Self-Healing & Diagnosis
From **failure** to **fixed test**: diagnosis-first repair, flaky detection, root cause analysis.

**Prompts here:**
- Diagnose why a test failed (assertion vs. flakiness vs. real bug)
- Suggest a fix to the test
- Detect & isolate flaky tests
- Decide: fix the test, fix the code, or investigate further?

### `/analyze` — Analysis & Trends
From **results** to **insight**: failure trends, test coverage, performance regression, risk exposure.

**Prompts here:**
- Summarize test results (pass rate, new failures, flaky tests)
- Triage failures by severity/area
- Detect performance regressions
- Recommend what to test next

---

## How to Use This Library

**If you're building a workflow:**
1. Start with `/plan` to create an oracle
2. Pick a `/author` prompt to scaffold tests
3. Use `/execute` to run them
4. Loop through `/heal` (fixing failures) and `/analyze` (understanding trends)

**If you're building a skill:**
1. Reference the relevant prompts in your SKILL.md
2. Adapt them for your specific language/framework
3. Link back to them in your documentation (don't copy/paste — link)

**If you're debugging a model's behavior:**
- Each prompt has a "Tradeoffs" section
- Explains what it trades for speed, cost, accuracy
- Helps you decide if it's right for your context

---

## Contributing a Prompt

See `/CONTRIBUTING.md` for the full guide. TL;DR:
1. Write the prompt in its relevant `/[stage]` folder
2. Add metadata: title, version, prerequisites, tradeoffs, example
3. Link to the concept it's grounded in
4. Submit a PR with explanation of why it's needed

---

*Last updated: Phase 0 — skeleton created*


# Oracle-First Workflow Orchestration

The complete pipeline: Requirement → Oracle → Tests → Execution → Report.

This is where `/prompts/`, `/mcp/`, and `/concepts/` come together into a real, executable workflow.

---

## The Five Phases (Revisited)

```
┌─────────────────────────────────────────────────────────────┐
│ PHASE 1: PLAN                                               │
│ Input: PRD, spec, or feature description                   │
│ Prompt: /prompts/plan/extract-oracle-from-prd.md           │
│ Output: Oracle (testable checklist)                         │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 2: AUTHOR                                             │
│ Input: Oracle + language/framework                          │
│ Prompt: /prompts/author/generate-tests-from-oracle.md      │
│ Output: Test code (functions, fixtures)                    │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 3: EXECUTE                                            │
│ Input: Test code + environment + MCP setup                 │
│ Prompt: /prompts/execute/run-tests-and-report.md           │
│ MCP: /mcp/cli-mcp-pattern.md or playwright-mcp-pattern.md  │
│ Output: Execution report (pass/fail + evidence)            │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 4: HEAL (Optional, if failures)                       │
│ Input: Failed tests + logs                                 │
│ Prompt: /prompts/heal/diagnose-test-failure.md             │
│ Output: Diagnosis + fix recommendation                     │
└──────────────────────────┬──────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ PHASE 5: ANALYZE                                            │
│ Input: Execution report + diagnoses                        │
│ Prompt: /prompts/analyze/summarize-results.md              │
│ Output: Readiness assessment (ship? defer? fix?)           │
└─────────────────────────────────────────────────────────────┘
```

---

## How to Run This Workflow

### Option A: Manual (Human + Claude)

1. **Plan:** Read sample-prd, ask Claude to extract oracle
2. **Author:** Give oracle to Claude, ask for test code
3. **Execute:** Set up MCP, have Claude run tests
4. **Heal:** If failures, ask Claude to diagnose
5. **Analyze:** Ask Claude for readiness assessment

This is flexible, interactive, good for learning.

### Option B: Scripted (Claude Agent Orchestration)

Create a Claude agent that:
1. Reads the PRD
2. Runs plan → author → execute → analyze in one go
3. Returns a final report

This is faster, less manual, good for CI/CD.

### Option C: Hybrid (Human Reviews, Claude Executes)

1. Human: Write the oracle (plan phase)
2. Claude: Author tests
3. Claude: Execute tests
4. Human: Review report, decide next steps

This balances control + efficiency.

---

## What's in This Directory

- **oracle-first-workflow.md** — How to orchestrate the full pipeline
- **sample-prd-walkthrough.md** — Complete walkthrough: password reset from PRD → report
- **/examples/** — Real, runnable examples
  - `password-reset-full-pipeline.md` — All 5 phases, end-to-end

---

## Starting Point: Use sample-prd-walkthrough.md

That document walks through the full password-reset example, showing:
- What the human does (write/review oracle)
- What Claude does (author, execute, analyze)
- Real prompts + responses
- Complete execution report
- Tradeoff analysis

Read it to understand the flow. Then adapt to your own PRD.

---

## Integration Points

- **Prompts:** Reference `/prompts/[stage]/` for each phase
- **MCP:** Reference `/mcp/[cli|playwright]-mcp-pattern.md` for execution
- **Concepts:** Ground decisions in `/concepts/oracle-executor-principle.md`
- **Tradeoffs:** Use `/guides/token-tradeoffs.md` to pick execution method

---

## Scaling This Workflow

**For one feature:** Use Option A (manual, interactive)

**For many features:** Use Option B (scripted agent) + human review gate

**For CI/CD:** Use Option B + automated allowlist/setup

**For learning:** Use Option A + read the guides alongside

---

## Example: Password Reset

See `/examples/sample-prd/` for:
- Original PRD
- Extracted oracle (plan phase)
- Generated test code (author phase)
- Execution report (execute phase)
- Final analysis (analyze phase)

---

*Phase 3: Workflow orchestration. The rubber meets the road.*


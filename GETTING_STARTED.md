# Getting Started with Oracle-First Testing

Welcome to the oracle-first testing framework! This guide will get you up and running quickly.

---

## Five Minutes: Understand the Core Idea

**The Oracle-Executor Principle**

```
Problem: LLMs miss bugs when they both write tests AND judge results
(because training data biases them toward "passing")

Solution: Split the work:
  - Oracle (human): What should we test? (checklist)
  - Executor (AI): Run those tests (report results)

Result: Models score 2x higher on defect detection
```

Read: [/concepts/oracle-executor-principle.md](concepts/oracle-executor-principle.md)

---

## 30 Minutes: See It In Action

Follow the complete password reset example:

1. **Phase 1 (Plan):** Extract oracle from PRD
2. **Phase 2 (Author):** Generate tests from oracle  
3. **Phase 3 (Execute):** Run tests, collect results
4. **Phase 4 (Heal):** Diagnose failures, fix them
5. **Phase 5 (Analyze):** Assess readiness

Read: [/workflow/sample-prd-walkthrough.md](workflow/sample-prd-walkthrough.md)

Time: 30 minutes to see the full loop end-to-end.

---

## Decide: Which Test Type Do You Need?

```
START: What are you testing?
├─ REST API? → Use CLI MCP (fast, cheap)
├─ Web UI? → Use Playwright MCP (interactive)
├─ Mobile app? → Use Appium (native elements)
├─ API contract? → Use Contract Testing (schema validation)
├─ Performance? → Use k6 or Locust (load testing)
└─ Something else? → See decision tree below
```

Full decision tree: [/guides/decision-tree-what-to-automate.md](guides/decision-tree-what-to-automate.md) (coming soon)

---

## Learn the Workflow

### Option A: Interactive (Learning)
Follow the prompts step-by-step with your own PRD.

Time: 90 minutes per feature
Tokens: 3-4k
Best for: Learning, first features

**Steps:**
1. Read [/prompts/plan/extract-oracle-from-prd.md](prompts/plan/extract-oracle-from-prd.md)
2. Give it your PRD → get oracle
3. Read [/prompts/author/generate-tests-from-oracle.md](prompts/author/generate-tests-from-oracle.md)
4. Give it oracle → get test code
5. Continue through execute → heal → analyze

### Option B: Automated (Production)
Use the scripted workflow for high throughput.

Time: 30 minutes per feature (with review gate)
Tokens: 3-4k
Best for: Many features, CI/CD

**Setup:**
```bash
# 1. Define oracle-first workflow (Python script or Claude agent)
./workflow orchestrate my-feature.md

# 2. Results:
# - tests/test_my_feature.py
# - results/execution.json
# - allure/index.html
```

See: [/workflow/oracle-first-workflow.md](workflow/oracle-first-workflow.md)

---

## Start Testing Your Own Feature

### Step 1: Write Your Oracle

Pick a feature (or copy password reset example). Create:

```markdown
# My Feature Oracle

## Happy Path (Critical)
- User can do X
- System responds with Y
- User can then do Z

## Edge Cases (High)
- What if input is empty?
- What if user is offline?
- What if this runs concurrently?

## Error Conditions (High)
- Invalid input → error message
- Service failure → graceful degradation

## Security (if applicable)
- No sensitive data in logs/errors
- Authorization is enforced
```

**Use:** [/prompts/plan/extract-oracle-from-prd.md](prompts/plan/extract-oracle-from-prd.md) to guide you.

### Step 2: Generate Tests

Give your oracle + language/framework to Claude:

```
Prompt: /prompts/author/generate-tests-from-oracle.md
Input: [Your oracle] + "Python + pytest"
Output: Test code
```

### Step 3: Run Tests

Set up MCP allowlist (safe endpoints), then execute:

```bash
pytest tests/test_my_feature.py -v --alluredir=allure-results
allure serve allure-results
```

### Step 4: Fix Failures

Use self-healing patterns:

```
Test fails → Run 10 times → Calculate pass rate
Pass rate 100%? → Healthy ✓
Pass rate <99%? → Flaky (use patterns from /self-healing/)
Pass rate 0%? → Real bug (use /prompts/heal/diagnose-test-failure.md)
```

### Step 5: Ship

Once tests pass + pass rate is stable, you're ready.

---

## Decision Tree: How to Test This?

```
What are you testing?

API endpoint?
  ├─ RESTful API → CLI MCP
  ├─ GraphQL → CLI MCP + GraphQL adapter
  └─ gRPC → CLI MCP + gRPC adapter

User interface?
  ├─ Web? → Playwright MCP
  ├─ Mobile? → Appium MCP
  └─ Desktop? → ??? (not yet in repo)

Multiple systems?
  ├─ Client/server contract? → Contract Testing
  ├─ Full end-to-end? → Playwright MCP + CLI MCP (hybrid)
  └─ Microservices? → Pact + contract testing

Non-functional?
  ├─ Performance? → k6, measure latency
  ├─ Load? → k6, Locust, concurrent users
  ├─ Security? → OWASP ZAP, injection tests
  └─ Resilience? → Chaos testing (Toxiproxy, mock failures)

Still not sure?
  └─ Read: /guides/decision-tree-what-to-automate.md
```

---

## Key Concepts Glossary

| Term | Meaning | See |
|------|---------|-----|
| **Oracle** | What to test (checklist from human) | [oracle-executor-principle.md](concepts/oracle-executor-principle.md) |
| **Executor** | AI running the tests (Claude + MCP) | [oracle-executor-principle.md](concepts/oracle-executor-principle.md) |
| **MCP** | Model Context Protocol (sandboxed tools) | [/mcp/README.md](mcp/README.md) |
| **Flaky test** | Fails intermittently (needs fixing) | [/self-healing/flaky-test-detection.md](self-healing/flaky-test-detection.md) |
| **Contract** | API promise (request/response schema) | [/tracks/contract-testing/README.md](tracks/contract-testing/README.md) |
| **NFR** | Non-functional requirement (SLA, load, security) | [/nfr/README.md](nfr/README.md) |

Full glossary: [/guides/glossary.md](guides/glossary.md) (coming soon)

---

## Common Patterns

### Pattern: Polling with Timeout
For async operations (email, notifications, etc.)

```python
def poll_until(check_func, max_wait=30):
    start = time.time()
    while time.time() - start < max_wait:
        result = check_func()
        if result:
            return result
        time.sleep(0.5)
    return None

# Usage:
email = poll_until(lambda: get_mailbox(), max_wait=30)
assert email is not None
```

See: [/self-healing/self-healing-patterns.md](self-healing/self-healing-patterns.md)

### Pattern: Test Isolation
Each test is independent (no pollution)

```python
def setup_method(self):
    self.test_id = str(uuid.uuid4())
    self.user_email = f"test-{self.test_id}@example.com"
    create_user(self.user_email, "password")

def teardown_method(self):
    delete_user(self.user_email)
```

### Pattern: Loose Assertions
Realistic expectations, not too strict

```python
# ✗ Don't: Too strict
assert response_time == 1.0

# ✓ Do: Realistic
assert response_time < 5.0  # SLA is 5 seconds
```

---

## Troubleshooting

### "Tests are flaky (sometimes pass, sometimes fail)"
→ Read: [/self-healing/flaky-test-detection.md](self-healing/flaky-test-detection.md)
→ Apply: [/self-healing/self-healing-patterns.md](self-healing/self-healing-patterns.md)

### "I don't know what to test"
→ Read: [/prompts/plan/extract-oracle-from-prd.md](prompts/plan/extract-oracle-from-prd.md)
→ Follow: [/workflow/sample-prd-walkthrough.md](workflow/sample-prd-walkthrough.md)

### "Tests are too expensive (too many tokens)"
→ Read: [/guides/token-tradeoffs.md](guides/token-tradeoffs.md)
→ Consider: Direct test execution for high-volume testing

### "My feature is already tested; can I integrate?"
→ Yes! Your existing tests are valid
→ Use Phase 4 (Heal) to find flaky tests
→ Use Phase 5 (Analyze) to assess coverage
→ Gradually adopt oracle-first for new features

---

## Next Steps

1. **Understand the principle** — Read [oracle-executor-principle.md](concepts/oracle-executor-principle.md) (5 min)
2. **See it in action** — Follow [sample-prd-walkthrough.md](workflow/sample-prd-walkthrough.md) (30 min)
3. **Test your own feature** — Write oracle, generate tests, run them (90 min)
4. **Integrate with team** — Set up CI/CD, Allure dashboards, ongoing monitoring

---

## Questions?

- **How do I contribute?** → See [CONTRIBUTING.md](CONTRIBUTING.md)
- **What's the research behind this?** → See [oracle-executor-principle.md](concepts/oracle-executor-principle.md)
- **Which tool should I use?** → See [token-tradeoffs.md](guides/token-tradeoffs.md)
- **I found a bug in the docs** → File an issue on GitHub

---

## Map of the Repo

```
Doctrines (WHY)
├── /concepts/oracle-executor-principle.md

Patterns (HOW)
├── /prompts/         (5 lifecycle stages)
├── /mcp/             (execution backends)
├── /self-healing/    (flaky detection + repair)
├── /reporting/       (JUnit + Allure)
└── /tracks/          (Appium, contracts, etc.)

Examples (SHOW ME)
├── /workflow/sample-prd-walkthrough.md
├── /mcp/examples/password-reset-cli-execution.md
└── /tracks/appium/examples/

Guides (HELP ME)
├── /guides/
├── GETTING_STARTED.md (you are here)
├── CONTRIBUTING.md
└── /guides/glossary.md
```

---

*Start with the 5-minute principle, graduate to the 30-minute example, then test your own feature.*


# Test Reporting & Metrics

Generate structured reports for test results, flaky detection, and trend analysis.

---

## Why Structured Reporting?

Raw test output is hard to parse and analyze:

```
PASSED test_user_can_reset_password
FAILED test_reset_email_arrives_within_30_seconds
PASSED test_old_password_doesn_t_work
...
```

Structured reports enable:
- **Visualization:** Pretty dashboards, graphs, trends
- **Analysis:** What's failing? What's flaky? Coverage?
- **Integration:** Feed results into other systems (CI/CD, dashboards)
- **History:** Track trends over time
- **Debugging:** Rich context (logs, screenshots, timing)

---

## Two Formats

### JUnit XML
For CI/CD integration, tool compatibility

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuites name="Password Reset Tests">
  <testsuite name="TestPasswordReset" tests="13" failures="0" time="42.3">
    <testcase name="test_user_can_reset_password" time="0.85">
      <!-- PASS -->
    </testcase>
    <testcase name="test_reset_email_arrives" time="2.3">
      <failure message="Timeout">Email not received after 30s</failure>
    </testcase>
  </testsuite>
</testsuites>
```

**When to use:**
- CI/CD integration (Jenkins, GitHub Actions, GitLab CI)
- Test report aggregation
- Historical tracking
- Cross-tool compatibility

---

### Allure Report
For rich visualization, detailed analysis

```
Allure Report:
├── Dashboard
│   ├── Pass rate: 85% (11/13 tests)
│   ├── Failures: 2 (flaky, real bug)
│   ├── Flakiness: 8% (1 test flaky)
│   └── Duration: 42 seconds
├── Tests (detailed view)
│   ├── test_user_can_reset_password (PASS, 0.85s)
│   ├── test_reset_email_arrives (FAIL, 2.3s)
│   │   └── Logs, artifacts, history
│   └── ...
├── Trends (over time)
│   ├── Pass rate trend
│   ├── Failure categories
│   └── Duration trend
└── Categories
    ├── Happy path: 5/6 PASS
    ├── Error handling: 5/5 PASS
    └── Edge cases: 1/2 PASS (1 flaky)
```

**When to use:**
- Team dashboards (see trends at a glance)
- Debugging failed tests (logs, history)
- Flaky test tracking (pass rate over time)
- Stakeholder reports (visual, easy to understand)

---

## What's in This Directory

- **junit-xml-format.md** — JUnit XML schema, examples, integration
- **allure-integration.md** — Allure report generation, categories, trends
- **/examples/** — Real report examples

---

## Recommended Setup

```
For each test run:
  1. Run tests (pytest, Jest, etc.)
  2. Generate JUnit XML (automatic in most frameworks)
  3. Generate Allure report (if desired)
  4. Upload both to CI/CD
  5. Publish JUnit to test reporter (Jenkins, GitHub)
  6. Publish Allure to static hosting or Allure TestOps
```

---

## Integration with Phase 3 & 4

**Phase 3 (Execute):** Generates raw test results  
**Reporting:** Transforms results into JUnit XML + Allure report  
**Phase 4 (Heal):** Uses reports to identify flaky tests  
**Phase 5 (Analyze):** Uses reports for readiness assessment

---

## Links

- **JUnit XML:** [junit-xml-format.md](junit-xml-format.md)
- **Allure Integration:** [allure-integration.md](allure-integration.md)
- **Phase 3 (Execute):** [/prompts/execute/run-tests-and-report.md](../prompts/execute/run-tests-and-report.md)
- **Phase 4 (Heal):** [self-healing/flaky-test-detection.md](../self-healing/flaky-test-detection.md)

---

*Phase 4: Reporting infrastructure. Measure what matters.*


# Self-Healing Framework

Diagnosis-first approach to flaky tests and automatic repair.

---

## The Problem

Tests fail. But why?

- **Real bug in code** → Fix the code
- **Flaky test** → Fix the test (race condition, timing, external service)
- **Test assumption changed** → Update the oracle or test
- **Environment issue** → Fix environment (not the test)

Most testers don't diagnose first. They just re-run and hope it passes. Bad signal.

The oracle-first approach says: **Always diagnose before fixing.**

---

## The Self-Healing Approach

```
┌──────────────────────────┐
│ Test Fails               │
└──────────────┬───────────┘
               ↓
┌──────────────────────────────────────────┐
│ Step 1: Collect Evidence                 │
│ - Run test 5+ times in isolation         │
│ - Gather pass rate, failure patterns     │
│ - Collect logs, timing data              │
│ - Check environment (API status, etc.)   │
└──────────────┬───────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│ Step 2: Diagnose Root Cause              │
│ - Is it deterministic or intermittent?   │
│ - Is the oracle ambiguous?               │
│ - Is it a timing issue? External service?│
│ - Is the code wrong?                     │
└──────────────┬───────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│ Step 3: Recommend Fix                    │
│ - Fix test: better wait, better assertion│
│ - Fix oracle: clarify ambiguous claim    │
│ - Fix code: implement missing behavior   │
│ - Fix environment: resolve dependency    │
└──────────────┬───────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│ Step 4: Apply Fix & Verify               │
│ - Apply the fix                          │
│ - Re-run test 10+ times                  │
│ - Verify fix didn't break anything else  │
└──────────────┬───────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│ Step 5: Root-Cause Analysis (Optional)   │
│ - Why did this happen?                   │
│ - How to prevent similar issues?         │
│ - Update test patterns/docs              │
└──────────────────────────────────────────┘
```

---

## Three Types of Failures

### Type 1: Deterministic Failures (100% fail rate)

**Pattern:** Test fails every time

**Diagnosis:**
- Code is broken
- Oracle is wrong
- Test assumption is outdated

**Fix:** Update code, oracle, or test (then verify it passes)

**Example:**
```
Test: test_user_can_reset_password
Result: 0/10 PASS (0% pass rate)
Diagnosis: Feature not implemented yet
Fix: Product team implements feature
```

---

### Type 2: Flaky Tests (Some pass, some fail)

**Pattern:** Test passes sometimes, fails other times (e.g., 60% pass rate)

**Diagnosis:**
- Race condition (test runs too fast, assertion happens before result)
- Timing issue (timeout too short for external service)
- External service instability (email service slow sometimes)
- Shared state (test pollutes environment for next test)

**Fix:** Make test more resilient (longer waits, better polling, cleaner setup)

**Example:**
```
Test: test_reset_email_arrives_within_30_seconds
Result: 6/10 PASS (60% pass rate)
Failures: 4/10 times, email takes >30s to arrive

Diagnosis: Email service in staging is slow/overloaded
Fix: Increase timeout to 45s, add exponential backoff polling
Re-run: 10/10 PASS (100%, email now arrives in avg 3.2s)
```

---

### Type 3: Environment Issues (Intermittent, Hard to Diagnose)

**Pattern:** Test passes in isolation, fails in CI, passes again later

**Diagnosis:**
- Test depends on shared resource (database, cache, etc.)
- Test order matters (test A pollutes state for test B)
- Infrastructure issue (network timeout, disk full)
- Test runs concurrently with others (race condition at infrastructure level)

**Fix:** Isolate test, add setup/teardown, increase timeouts, check infrastructure

**Example:**
```
Test: test_user_can_login
Result: Passes locally (100%), fails in CI (20% of runs)

Diagnosis: CI database is shared across parallel test runs
  Test A: Creates user "test@example.com", logs in
  Test B: Also creates "test@example.com", conflicts
Fix: Randomize test email (test-{uuid}@example.com)
Re-run in CI: 100% PASS (100 concurrent runs)
```

---

## What's in This Directory

- **flaky-test-detection.md** — How to detect flaky tests (patterns, severity levels)
- **self-healing-patterns.md** — Common patterns and fixes for flaky tests
- **/examples/** — Real examples (race conditions, timing, external services)

---

## Integration with Phases

**Phase 3 (Execute):** Tests fail → collect results  
**Phase 4 (Heal) + Self-Healing:** Diagnose failures, recommend fixes  
**Phase 5 (Analyze):** Assess readiness (is flakiness acceptable?)

---

## Success Criteria

A test is **healthy** if:
- ✓ It passes 100% of the time when run in isolation
- ✓ It passes 99%+ of the time in CI/CD (accounting for external service SLAs)
- ✓ Failures are deterministic (always fail for the same reason, always pass when fixed)

A test is **flaky** (needs repair) if:
- ✗ It passes <99% of the time
- ✗ Failures are intermittent and unpredictable
- ✗ Pass rate varies by environment (passes locally, fails in CI)

---

## When to Ignore Flakiness

Some flakiness is acceptable IF:
- The external service has SLA <99% (e.g., test email delivery)
- The test is low-priority (monitoring only, not blocking)
- The flakiness is rare (<1% fail rate) and low-impact

**Document these cases.** Don't sweep flakiness under the rug without acknowledgment.

---

## Automation Possibilities

You can automate self-healing:

1. **Continuous monitoring:** Run tests every hour, track pass rates
2. **Automatic flaky detection:** If pass rate <99%, flag for diagnosis
3. **Suggested fixes:** AI can recommend fixes (longer timeouts, retry logic, etc.)
4. **Automatic retry:** Re-run flaky tests N times, take majority vote
5. **Alerting:** Notify team if flakiness increases (regression)

See `/reporting/` for integration with test report formats.

---

## Links

- **Heal Prompt:** [/prompts/heal/diagnose-test-failure.md](../prompts/heal/diagnose-test-failure.md)
- **Analyze Prompt:** [/prompts/analyze/summarize-results.md](../prompts/analyze/summarize-results.md)
- **Flaky Detection:** [flaky-test-detection.md](flaky-test-detection.md)
- **Patterns & Fixes:** [self-healing-patterns.md](self-healing-patterns.md)

---

*Phase 4: Self-healing foundation. Fix tests, not symptoms.*


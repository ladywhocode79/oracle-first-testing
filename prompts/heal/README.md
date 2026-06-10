# Heal: Diagnosis-First Test Repair

When a test fails, diagnose why and fix it. Distinguish between flaky tests, real bugs, and broken oracles.

## What Goes Here

Prompts for:
- Diagnosing test failures (assertion failure vs. flakiness vs. timeout vs. real bug)
- Suggesting fixes to broken tests
- Detecting flaky tests (multiple runs, statistical methods)
- Analyzing logs to understand the root cause
- Deciding: fix the test, fix the code, fix the oracle, or investigate further?
- Running a failing test in isolation with extra logging

## Input

- A failing test result (from `/execute`)
- Optional: logs, stack traces, prior execution history
- The oracle (to check if the test was even checking the right thing)

## Output Goal

A **diagnosis** and **recommendation**:

```
Test: test_reset_link_expires_after_24h
Status: FAIL (assertion error)

Diagnosis:
  Root cause: Test assumes link expires exactly at 24h. 
  Real behavior: Link expires after 24h +/- 60s (clock skew).
  Verdict: Broken test (oracle too strict), not a real bug.

Recommendation:
  Fix the test: allow 24h ±60s window.
  Priority: Medium (affects reliability, not correctness).
```

Another example:

```
Test: test_password_reset_email_delivered
Status: FAIL (3/10 runs, flaky)

Diagnosis:
  Root cause: Email delivery is eventually consistent. 
  1s poll window too narrow for SLA (30s).
  Verdict: Flaky test, not a real bug.

Recommendation:
  Fix the test: increase poll window to 30s, add exponential backoff.
  Priority: High (unreliable signal, blocks CI).
```

## Key Principle

**Diagnosis first.** Don't assume a failure is a real bug. The agent diagnoses, the human decides what to do.

---

*Last updated: Phase 0 — skeleton created*


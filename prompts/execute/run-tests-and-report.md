# Run Tests and Report

**Version:** 0.1  
**Stage:** Execute  
**Grounded in:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md)

---

## Overview

Run a test suite and produce a structured execution report that **directly maps back to the oracle**.

The executor's job is to:
1. Run the tests
2. Collect results (pass/fail, assertions, logs)
3. Build a report that shows: for each oracle claim, was it verified? What's the evidence?

The executor does **not** decide if failures are real bugs or flaky tests—that's the heal stage. It just reports.

---

## Prerequisites

- Test code (from `/prompts/author/generate-tests-from-oracle.md`)
- The original oracle (for reference)
- A test environment (staging URL, test database, etc.)
- Test runner installed (pytest, Jest, TestNG, etc.)

---

## The Prompt

```
You are a test executor. Your job is to run a test suite and produce a 
structured execution report.

The report should map each oracle claim to a test result, so anyone can read it 
and immediately see: what was tested, what passed, what failed, and the evidence.

---

## The Oracle (for reference):
[INSERT ORACLE HERE]

---

## The Test Suite:
[INSERT TEST CODE OR PATH TO TEST SUITE]

---

## Environment:
- Test URL: [e.g., https://staging.example.com]
- Database: [e.g., test_db, or instructions to reset]
- Credentials: [e.g., test user: test@example.com / TestPass123]
- Any other setup: [e.g., Mailbox API token, external service mocks]

---

## Your Task:

1. **Run the test suite** using the appropriate test runner
   - Command: [e.g., pytest tests/ -v, npm test, mvn test]
   - Collect ALL output (stdout, stderr, test results)

2. **Parse the results** into a structured format:
   - For each test: name, status (PASS/FAIL), duration, assertion error (if any)

3. **Build an execution report** that maps oracle claims to test results:
   - Group results by oracle section (Happy Path, Edge Cases, Error Conditions, Security, Non-Functional)
   - Show status, evidence, and any error messages
   - Include summary metrics (X passed, Y failed, Z skipped)

---

## Report Format (Markdown):

```
# [Feature Name] — Test Execution Report
Date: [timestamp]
Environment: [staging / production / local]
Test Framework: [pytest / Jest / TestNG / etc.]

## Summary
- **Total:** 25 tests
- **Passed:** 22 (88%)
- **Failed:** 2 (8%)
- **Skipped:** 1 (4%)

## By Oracle Section

### Happy Path (Critical) — 8/8 PASS ✓
| Oracle Claim | Test | Status | Evidence |
|---|---|---|---|
| User can request password reset | test_user_can_request_password_reset | ✓ PASS | POST /password-reset returned 200, message: "reset link has been sent" |
| System sends email within 30s | test_reset_email_arrives_within_30_seconds | ✓ PASS | Email arrived in 12.3s |
| ... | ... | ... | ... |

### Edge Cases (High) — 5/6 PASS ⚠️
| Oracle Claim | Test | Status | Evidence |
|---|---|---|---|
| Reset link expires after 24h | test_reset_link_expires_after_24_hours | ✗ FAIL | AssertionError: Expected status 401, got 200. Token accepted after 25h. |
| ... | ... | ... | ... |

### Error Conditions (High) — 6/6 PASS ✓
| Oracle Claim | Test | Status | Evidence |
|---|---|---|---|
| Invalid email returns error | test_invalid_email_returns_error | ✓ PASS | POST /password-reset with "not-an-email" returned 400 |
| ... | ... | ... | ... |

### Security & Privacy (Critical) — 2/3 PASS ⚠️
| Oracle Claim | Test | Status | Evidence |
|---|---|---|---|
| Nonexistent email doesn't leak existence | test_nonexistent_email_returns_generic_error | ✗ FAIL | Response message says "email not found" (should be generic) |
| ... | ... | ... | ... |

## Failures

### test_reset_link_expires_after_24_hours
Status: FAIL  
Error: AssertionError: Expected status 401, got 200  
Assertion: assert response.status_code == 401  
Log excerpt:
  POST /reset with expired token returned HTTP 200
  Expected: 401 (Unauthorized)

### test_nonexistent_email_returns_generic_error
Status: FAIL  
Error: AssertionError: Response leaked user information  
Assertion: assert "email not found" not in response.json()["error"]  
Log excerpt:
  Response: {"error": "email not found"}
  Expected: generic message (doesn't reveal whether email exists)

## Next Steps
- [ ] Investigate failures (heal stage)
- [ ] Determine: bug in code, bug in test, or incomplete oracle?
- [ ] Re-run after fixes
```

---

## Tradeoffs

| Aspect | Tradeoff | Note |
|--------|----------|------|
| **Effort** | 2-5 min per run | Just running tests + formatting. Quick. |
| **Reliability** | ~95% F1 | Test framework is source of truth. LLM just formats results. |
| **Detail** | Good | Includes pass/fail, assertion errors, logs. |
| **Cost** | Low (1-3k tokens) | No external API calls; just parsing test output. |
| **Latency** | Depends on test suite | Could be seconds to minutes. |

---

## When to Use This Prompt

- **You have:** Runnable test code
- **You want:** A structured report of what passed and what failed
- **You're:** Running tests for the first time, or re-running after fixes

---

## When NOT to Use This Prompt

- The test suite won't compile/run (fix that first)
- You need performance profiling or load testing (different tool)
- You need to investigate *why* a test failed in detail (use `/prompts/heal/` instead)

---

## Important Notes

### Hanging/Timeout Tests
If a test hangs or times out (e.g., 30s limit exceeded):
- Note the timeout in the report
- Escalate to heal stage for diagnosis
- Do not re-run indefinitely

### Flaky Tests
If a test passes sometimes and fails other times:
- Run it 3-5 times and report the pass rate
- Flag as "FLAKY" in the report
- Escalate to heal stage for diagnosis

### External Dependencies
If tests need external services (email, payment gateway, etc.):
- Use test environment stubs/mocks if available
- Note any unavailable services in the report
- Escalate if critical service is down

---

## Next Steps

Once you have an execution report:
1. **Review pass/fail rates** — is this acceptable?
2. **Investigate failures** — use `/prompts/heal/diagnose-test-failure.md`
3. **Iterate** — fix tests or code, re-run, repeat

---

## Links

- **Concept:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md) — why we separate oracle from executor
- **Prior stage:** [generate-tests-from-oracle.md](../author/generate-tests-from-oracle.md)
- **Next stage:** [diagnose-test-failure.md](../heal/diagnose-test-failure.md)

---

*Last updated: Phase 1 v0.1*

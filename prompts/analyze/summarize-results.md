# Summarize Results & Recommend Next Steps

**Version:** 0.1  
**Stage:** Analyze  
**Grounded in:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md)

---

## Overview

Turn test results into actionable insight:
- What's the health of this feature?
- What failures matter?
- What should we fix or test next?
- Are we ready to ship?

---

## Prerequisites

- Execution report (from `/prompts/execute/run-tests-and-report.md`)
- Diagnoses of failures (from `/prompts/heal/diagnose-test-failure.md`)
- The original oracle
- Context: is this a new feature, a regression, a hot fix?

---

## The Prompt

```
You are a test analyst. Your job is to read test results and tell the human tester:
- What's working
- What's broken
- What the real risks are
- What to do next

---

## The Execution Report:
[INSERT TEST RESULTS SUMMARY]

Example:
  Total: 25 tests
  Passed: 22 (88%)
  Failed: 2 (8%)
  Skipped: 1 (4%)
  
  Failures:
    - test_reset_link_expires_after_24_hours (flaky oracle, not a bug)
    - test_nonexistent_email_returns_generic_error (real security bug)

---

## The Oracle:
[INSERT ORACLE]

---

## Context:
[e.g., "New feature, first test run", "Regression, used to pass", "Hot fix for production bug"]

---

## Your Task:

### Step 1: Triage Results
Separate:
- **Pass:** Confidence ✓ (working as specified)
- **Flaky:** Noise (intermittent, needs investigation)
- **Real failures:** These matter (real bugs, security issues, incomplete oracle)

Group by severity:
- **Critical:** Blocks launch, affects core functionality, security/privacy issue
- **High:** Affects user experience, important feature broken, compliance risk
- **Medium:** Edge case broken, performance degraded, "nice to have" missing
- **Low:** Cosmetic, documentation issue, nice-to-have feature incomplete

### Step 2: Assess Risk
For each critical/high failure, ask:
- Does this affect users? How many? What happens to them?
- Is this a blocker for launch? Why or why not?
- Is it a regression (worked before, broken now)? If so, what changed?

### Step 3: Recommend Actions
For each failure:
- **Fix immediately:** (critical bugs, security issues)
- **Fix this cycle:** (high-priority, affects this feature)
- **Defer:** (nice-to-have, can fix later)
- **Monitor:** (flaky but eventually works, watch for patterns)

### Step 4: Coverage Analysis
Ask:
- Are all critical paths tested? (Happy path + major edge cases)
- Are security-sensitive areas tested? (Auth, data access, sensitive data)
- Are error cases tested? (Invalid input, timeouts, external failures)
- What's untested that should be tested?

### Step 5: Readiness Assessment
Can we ship this feature? (Yes / No / With caveats)
- If YES: Why is it safe?
- If NO: What blockers remain?
- If caveats: What risks are we accepting?

---

## Output Format:

```
# [Feature Name] — Test Analysis Report

## Health Summary
- **Status:** ✓ Ready to ship / ⚠️ Conditional / ✗ Blockers remain
- **Pass rate:** 88% (22/25)
- **Risk level:** Low / Medium / High / Critical
- **Key concern:** [One sentence summary of biggest issue]

## Triage

### Critical Issues (Block Launch) ❌
| Issue | Test | Impact | Fix Priority |
|-------|------|--------|---|
| Security: email enumeration | test_nonexistent_email_returns_generic_error | Users can be identified; phishing vector | Fix immediately |

### High Priority (Address This Cycle) ⚠️
| Issue | Test | Impact | Fix Priority |
|-------|------|--------|---|
| Ambiguous oracle: 24h expiry | test_reset_link_expires_after_24_hours | Test is flaky; hard to trust results | Fix oracle clarity, then test |

### Medium Priority (Monitor) 📋
| Issue | Test | Impact | Fix Priority |
|-------|------|--------|---|
| No tests for concurrent resets | N/A | Edge case; low user impact | Add test after launch |

### Low Priority (Nice to Have) 💡
| Issue | Test | Impact | Fix Priority |
|-------|------|--------|---|
| No load testing | N/A | Unknown scaling limits | Monitor, plan for Phase 2 |

## Coverage Analysis

### ✓ Well Covered
- Happy path (user requests reset, gets email, sets new password)
- Error handling (invalid email, expired link)
- Basic security (token doesn't leak in logs)

### ⚠️ Partially Covered
- Edge cases (concurrent requests, timing edge cases)
- Security depth (CSRF protection, rate limiting, account enumeration)

### ✗ Not Covered
- Load testing (how many concurrent resets can the system handle?)
- Performance regression (is it slower than before?)
- Integration (does password reset work with SSO / LDAP / etc.?)
- Admin features (can admins force a reset? audit trail?)

### Recommendation
- Required before launch: Fix email enumeration vulnerability
- Nice to have: Add 3-4 tests for concurrent/timing edge cases
- Plan for Phase 2: Load testing, SSO integration, admin audit trail

## Readiness Assessment

**Can we ship?** ✓ **Yes, with critical fix**

**Why:**
- Core functionality is solid (8/8 happy path tests pass)
- Error handling works (6/6 error case tests pass)
- Only real issue is security: email enumeration vulnerability

**Blockers:**
- [ ] Fix: Change error messages to be generic (don't say "email not found")
- [ ] Test: Verify fix passes test_nonexistent_email_returns_generic_error
- [ ] Security review: Are there other enumeration vectors?

**Known risks:**
- Test flakiness on 24h expiry timing (oracle too strict, not a real product issue)
- No concurrent-reset testing (assumed rare, can monitor in production)
- No load testing (SLA untested; assume 100 reqs/sec, scale later)

**Recommendation:**
1. Fix security issue (1-2 hours)
2. Re-run tests (10 min)
3. Ship
4. Monitor flakiness in staging; add concurrent test if needed

## Metrics

| Metric | Value | Healthy? |
|--------|-------|----------|
| Pass rate | 88% | ⚠️ Good, but failures are concerning |
| Flaky tests | 1/25 (4%) | ✓ Acceptable |
| Critical failures | 1/25 (4%) | ✗ Must fix before launch |
| High failures | 1/25 (4%) | ⚠️ Address before launch |
| Coverage (happy path) | 100% | ✓ Excellent |
| Coverage (edge cases) | 60% | ⚠️ Good enough for MVP |
| Coverage (security) | 80% | ✓ Good, but enumeration gap |

## Next Steps

**Immediate (before launch):**
1. [ ] Fix password reset endpoint to return generic error messages
2. [ ] Verify test_nonexistent_email_returns_generic_error passes
3. [ ] Run full test suite once more; expect 25/25 PASS
4. [ ] Security team review the fix
5. [ ] Deploy to production

**This sprint (after launch):**
1. [ ] Monitor production for any reset failures
2. [ ] Investigate test flakiness on 24h expiry; may need more sophisticated test
3. [ ] Add tests for concurrent reset requests (if user reports indicate race conditions)

**Future (Phase 2):**
1. [ ] Add load testing (k6 / Gatling)
2. [ ] Add performance regression tests
3. [ ] Expand coverage: SSO, LDAP, admin audit trail, webhooks
4. [ ] Chaos testing: what if email service is down? database slow?
```

---

## Tradeoffs

| Aspect | Tradeoff | Note |
|--------|----------|------|
| **Effort** | 15-30 min | Depends on number of failures and need for context. |
| **Accuracy** | ~80-90% F1 | LLM-based analysis; always review before shipping. |
| **Depth** | Good | Covers triage, risk assessment, coverage gaps, readiness. |
| **Cost** | Medium (5k tokens) | Includes summary, triage, analysis, and recommendations. |

---

## When to Use This Prompt

- **You have:** Execution report + diagnoses of failures
- **You want:** To decide if this feature is ready to ship
- **You're:** Ready to ship, need risk assessment

---

## When NOT to Use This Prompt

- Tests are still failing and not diagnosed (run heal stage first)
- You don't have a clear oracle (ambiguity will muddy analysis)
- You're just debugging one specific failure (use heal stage instead)

---

## Important Notes

### Readiness ≠ Pass Rate
A 100% pass rate doesn't mean you're ready to ship. If coverage is incomplete, you're just not testing the right things.

Conversely, 80% pass rate is fine IF:
- The failures are flaky (not real bugs)
- Or failures are low-priority (edge cases, future features)
- And critical paths are covered

### Risk is About Impact, Not Likelihood
A rare security bug (0.1% of users hit it) is still critical.
A cosmetic regression (100% of users see it) is lower priority.

### Default to Caution on Security
If there's a security issue, fix it before shipping. Don't defer.

---

## Next Steps

Based on the analysis:
1. **If "Ready to ship":** Deploy with confidence
2. **If "Conditional":** Fix blockers, re-analyze, then ship
3. **If "Blockers remain":** Go back to plan/author/execute loop, fix tests or code, re-run

---

## Links

- **Concept:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md)
- **Prior stage:** [diagnose-test-failure.md](../heal/diagnose-test-failure.md)
- **Full lifecycle:** [Plan](../plan/extract-oracle-from-prd.md) → [Author](../author/generate-tests-from-oracle.md) → [Execute](../execute/run-tests-and-report.md) → [Heal](../heal/diagnose-test-failure.md) → [Analyze](../analyze/summarize-results.md)

---

*Last updated: Phase 1 v0.1*

# Diagnose Test Failure

**Version:** 0.1  
**Stage:** Heal  
**Grounded in:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md)

---

## Overview

When a test fails, diagnose **why**. Is it:
- **Broken test** — the test is wrong, the oracle was incomplete, or assumptions changed?
- **Flaky test** — the test sometimes passes, sometimes fails (timing, external service, race condition)?
- **Real bug** — the code doesn't match the oracle?

Once you diagnose, recommend a fix. But **diagnosis always comes first**.

---

## Prerequisites

- A failing test result (from `/prompts/execute/run-tests-and-report.md`)
- The test code
- The original oracle
- Logs, stack traces, or error messages
- Access to the environment where the test failed

---

## The Prompt

```
You are a test diagnostician. Your job is to figure out WHY a test failed and 
recommend a fix.

Many failures are NOT real bugs. They're broken tests, flaky assertions, or 
incomplete oracles. Your job is to distinguish.

---

## The Failure:
[INSERT TEST NAME, STATUS, AND ERROR MESSAGE]

Example:
  Test: test_reset_link_expires_after_24_hours
  Status: FAIL
  Error: AssertionError: Expected status 401, got 200
  Log: POST /reset with token=abc123 (created 25h ago) returned HTTP 200

---

## The Test Code:
[INSERT THE TEST FUNCTION HERE]

---

## The Oracle Claim:
[INSERT THE CLAIM THIS TEST IS VERIFYING]

Example:
  "Reset link expires exactly 24 hours after being issued (request not allowed after)"

---

## The Environment:
- Test URL: [e.g., https://staging.example.com]
- Last code change: [e.g., "password reset PR merged 3h ago", or "no recent changes"]
- Is this test flaky? [e.g., "first failure", "fails ~50% of the time", "always fails"]
- Can you reproduce it? [Can you run the test again and see if it fails again?]

---

## Your Task:

### Step 1: Understand the Failure
- What was the test trying to verify?
- What did it expect to happen?
- What actually happened?
- What's the gap?

### Step 2: Identify Root Cause
Ask:
- Is the oracle claim ambiguous or incomplete? (Oracle problem)
  - Example: "expires after 24h" — does that mean exactly at 24h00:00? Or 24h ± some tolerance?
  - Example: "email arrives within 30s" — which 30s? User perceives, or system-measured?
- Is the test implementation wrong? (Test problem)
  - Example: Test creates token at time T, checks at T+24h. But what if there's clock skew?
  - Example: Test asserts response.status_code == 401, but API returns 403 (semantically correct, wrong code)
  - Example: Test tries to poll for email, but mailbox API is flaky
- Is the code wrong? (Product problem)
  - Example: Reset token doesn't actually expire
  - Example: Email service is broken for this user
  - Example: Database clock is 6 hours behind
- Is it flaky (environment problem)?
  - Example: Same test fails ~60% of the time (suggests race condition or external service latency)
  - Example: Test passes in local env but fails in staging (suggests environment misconfiguration)

### Step 3: Gather Evidence
- Can you run the test in isolation? Does it still fail?
- Can you add logging to see what's happening?
- Can you reproduce the failure manually (e.g., via curl or UI)?
- Does the failure happen consistently or intermittently?

### Step 4: Recommend a Fix
Based on diagnosis, recommend ONE of:

**Option A: Fix the Oracle**
- If the claim is ambiguous
- What should the claim be instead?
- Impact: Affects all related tests

**Option B: Fix the Test**
- If the test implementation is wrong
- What exactly needs to change?
- Impact: Only this test, quick fix

**Option C: Fix the Code**
- If the oracle is correct and the test is correct, but the code doesn't match
- What should the code do?
- Impact: Product change, affects multiple tests possibly

**Option D: It's Flaky (Needs Diagnosis, Not Immediate Fix)**
- If the test fails intermittently
- What could be causing the race condition or timing issue?
- Recommendation: Run test 10 times in a loop, collect pass rate and failure patterns
- Then revisit with more evidence

---

## Output Format:

```
# Diagnosis: [Test Name]

## The Failure
- Expected: [what the test expected]
- Actual: [what actually happened]
- Gap: [the mismatch]

## Root Cause
**Type:** [Oracle / Test / Product / Flaky]

**Analysis:**
[Detailed explanation of why this happened]

**Evidence:**
- [Evidence point 1: e.g., "Reset token created at 12:00:00, checked at 12:00:01+24h. System clock: 12:00:05+24h (5s skew)."]
- [Evidence point 2]

## Recommendation

**Action:** [Fix Oracle / Fix Test / Fix Product / Run Stability Check]

**Details:**
[Specific changes needed. If fixing oracle: new claim. If fixing test: code changes. If fixing product: what needs to change. If flaky: how to diagnose further.]

**Priority:** [Critical / High / Medium / Low]
- Why: [e.g., "Oracle was too strict; test is flaky but functional", "Blocks users", "Edge case only"]

## Next Steps
1. [Step 1]
2. [Step 2]
3. Re-run test to confirm fix
```

---

## Example 1: Broken Oracle

```
# Diagnosis: test_reset_link_expires_after_24_hours

## The Failure
- Expected: POST with expired token returns 401 (Unauthorized)
- Actual: POST with expired token returns 200 (OK, password was reset)
- Gap: Token was not rejected as expected

## Root Cause
**Type:** Oracle (incomplete claim)

**Analysis:**
The oracle says "reset link expires after 24 hours." But it doesn't define the tolerance. 
The test assumes EXACTLY 24h00m00s expiry. However:
1. The system allows ±60s clock skew (system design)
2. The test creates token at T, checks at T+24h+0.001s (clock skew in test framework)
3. Result: Token is still valid, even though "24 hours" has technically passed

The oracle claim is ambiguous. "Expires after 24h" could mean:
- Expires at exactly 24:00:00 (not realistic, given clock skew)
- Expires within [24:00:00 - 1min, 24:00:00 + 1min] (realistic)
- Expires after 24h, not before (current implementation is correct)

## Recommendation

**Action:** Fix Oracle

**New claim:**
"Reset link is valid for at least 24 hours, and invalid after 24 hours + 5 minutes (to account for clock skew and system tolerance)"

**Details:**
- Old: "expires exactly 24 hours"
- New: "expires within [24h, 24h+5m]"
- Rationale: Systems can't guarantee exact second-level expiry; build in tolerance

**Priority:** Medium
- This is not a security bug (5m extra window is acceptable)
- But it does cause test flakiness and confusion

## Next Steps
1. Update the oracle claim in the test suite documentation
2. Update test to use a 24h+5m window: `assert elapsed <= 24*60*60 + 5*60`
3. Re-run; should now PASS
4. Monitor: if it still flakes, investigate clock skew further
```

---

## Example 2: Broken Test

```
# Diagnosis: test_nonexistent_email_returns_generic_error

## The Failure
- Expected: Response should NOT contain "email not found"
- Actual: Response contains {"error": "email not found"}
- Gap: User information leaked (security issue)

## Root Cause
**Type:** Test (or Test + Product)

**Analysis:**
The test is correct. The oracle claim is correct: "Nonexistent email does not reveal whether 
email exists (security)."

But the code is returning "email not found," which DOES reveal that the email wasn't registered.

This is a real bug, not a test problem. The test caught it.

## Recommendation

**Action:** Fix Product

**Details:**
The endpoint currently returns specific error messages:
  - "email not found" (user enumeration vulnerability)
  - "password is incorrect" (user enumeration, phishing aid)
  - "account is locked" (account enumeration)

Security best practice: Return generic message for all authentication/reset failures:
  - Change response to: {"message": "If that email is registered, a reset link has been sent"}
  - Same message for both existing and non-existing emails
  - Same message for all errors (rate limit exceeded, service down, etc.)

Rationale: Prevents user enumeration attacks

**Priority:** Critical
- This is a security vulnerability (user enumeration)
- Must be fixed before launch
- Affects password reset, login, and other auth endpoints

## Next Steps
1. Update the endpoint to return generic messages
2. Update any other endpoints with similar issues
3. Re-run test; should now PASS
4. Security review: are there other user enumeration risks?
```

---

## Example 3: Flaky Test

```
# Diagnosis: test_reset_email_arrives_within_30_seconds

## The Failure
- Status: FLAKY (passes 3/10 times, fails 7/10 times)
- Failure: AssertionError: Email did not arrive (polled for 30s, still not in mailbox)
- Gap: Intermittent email delivery delays

## Root Cause
**Type:** Flaky (external service latency)

**Analysis:**
The test polls the mailbox API every 1 second for up to 30 seconds. 
This is correct. But email delivery in staging is inconsistent:
- Sometimes arrives in <1s (most of the time)
- Sometimes takes 5-15s (occasional, under load)
- Rarely takes 20-30s (external service degradation)

The oracle claim says "arrives within 30 seconds" — which is correct. But the test 
only polls every 1 second, so it might miss the delivery if it happens between polls.

More likely: the external email service (Sendgrid / AWS SES / etc.) is occasionally slow, 
and the 30s timeout is too tight for the test environment.

## Recommendation

**Action:** Run Stability Check (diagnose flakiness further)

**Details:**
This is not a bug in the code, oracle, or test—it's environmental inconsistency. 
Before fixing, gather more data:

1. Run the test 10-20 times and collect:
   - How many pass vs fail?
   - What's the median/p95/p99 delivery time?
   - Does it correlate with time of day or load?

2. Check email service health:
   - Is Sendgrid/SES experiencing issues?
   - Is staging overloaded?
   - Is the mailbox API slow?

3. If the oracle (30s) is realistic but the test is flaky due to environment:
   - Increase poll frequency (every 0.5s instead of 1s)
   - Increase timeout (to 45s)? No—if 30s is the real SLA, don't hide it
   - Or: Check if this is acceptable flakiness (e.g., OK for "test" but not "CI blocker")

**Priority:** Medium
- Flaky tests are noise (unreliable signals)
- But if email *actually* arrives in <30s, the test will eventually pass
- If it's *really* slow, that's a product issue, not a test issue

## Next Steps
1. Run test 10 times in a loop: `pytest tests/test_reset.py::test_reset_email_arrives_within_30_seconds -v --count=10`
2. Collect pass rate and timing distribution
3. Investigate email service (check logs, alert thresholds)
4. Decide: increase timeout, improve test polling, or escalate as product issue?
```

---

## Tradeoffs

| Aspect | Tradeoff | Note |
|--------|----------|------|
| **Effort** | 10-30 min per failure | Depends on complexity. Simple failures (wrong assertion) = quick. Flaky failures = longer. |
| **Accuracy** | ~70-80% first-time | Diagnosis is human judgment. Always review. |
| **Cost** | Medium (3-5k tokens) | May need to re-run tests, collect logs, etc. |
| **Latency** | Depends on debugging | Quick if cause is obvious; slow if need to gather evidence. |

---

## When to Use This Prompt

- **You have:** A failing test
- **You want:** To understand why it failed
- **You need:** A recommendation for fixing it

---

## When NOT to Use This Prompt

- The test suite doesn't run at all (fix compilation errors first)
- You don't have access to logs or error details (gather those first)
- The oracle is completely vague (refine the oracle first)

---

## Important Notes

### Always Prioritize Diagnosis Over Guessing
Don't assume "the code is broken." Diagnose first. Many failures are:
- Broken tests (70% of cases)
- Incomplete oracles (20%)
- Real bugs (10%)

### Document Findings
Whatever you diagnose, document it. Next time you see a similar failure, you'll save time.

### Flaky Tests Are a Signal
Don't ignore them. They're telling you something is unstable (code, test, or environment). 
Investigate.

---

## Next Steps

Once you have a diagnosis and recommendation:
1. **Implement the fix** (oracle, test, or product code)
2. **Re-run the test** (via `/prompts/execute/run-tests-and-report.md`)
3. **If it passes:** move to analyze stage
4. **If it still fails:** escalate for human review

---

## Links

- **Concept:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md) — why separating oracle from executor helps here
- **Prior stage:** [run-tests-and-report.md](../execute/run-tests-and-report.md)
- **Next stage:** [summarize-results.md](../analyze/summarize-results.md)

---

*Last updated: Phase 1 v0.1*

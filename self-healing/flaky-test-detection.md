# Flaky Test Detection

How to identify, measure, and categorize flaky tests.

---

## What Is a Flaky Test?

A test is **flaky** if it passes sometimes and fails other times, even when the code hasn't changed.

```
Deterministic test:
  Run 1: PASS
  Run 2: PASS
  Run 3: PASS
  ← Reliable signal

Flaky test:
  Run 1: PASS
  Run 2: FAIL (same code, same environment)
  Run 3: PASS
  ← Unreliable signal (useless for CI)
```

---

## Detection Method 1: Run in Isolation

The simplest way to detect flakiness:

```bash
# Run the same test 10 times in a row
pytest tests/test_password_reset.py::test_reset_email_arrives_within_30_seconds -v --count=10

# Results:
# PASSED [10%]
# PASSED [20%]
# PASSED [30%]
# FAILED [40%]  ← Flaky!
# PASSED [50%]
# PASSED [60%]
# FAILED [70%]  ← Flaky!
# PASSED [80%]
# PASSED [90%]
# PASSED [100%]

# Pass rate: 8/10 (80%) → FLAKY
```

**Severity:**
- 100% pass: Healthy ✓
- 99%+ pass: Acceptable (account for SLA)
- 95-99% pass: Flaky (needs fix)
- <95% pass: Very flaky (urgent fix)

---

## Detection Method 2: Track in CI/CD

Monitor test results over time in your CI pipeline:

```
Test: test_reset_email_arrives_within_30_seconds

Week 1: 10/10 PASS (100%) ✓
Week 2: 9/10 PASS (90%) ⚠️ Trend detected
Week 3: 8/10 PASS (80%) ⚠️ Getting worse
Week 4: 6/10 PASS (60%) ✗ Now flaky

Action: Investigate regression. What changed in Week 2?
```

**Tools:**
- JUnit XML + Allure Report (see `/reporting/`)
- Custom monitoring dashboard
- CI logs + grep/analyze

---

## Detection Method 3: Statistical Analysis

For tests run many times, calculate:

```
Pass rate = (# passed) / (# run)
Standard deviation = sqrt(pass_rate * (1 - pass_rate) / n)
Confidence interval = pass_rate ± (1.96 * stdev)

Example:
  50 runs, 40 passed, 10 failed
  Pass rate = 80%
  Stdev = sqrt(0.8 * 0.2 / 50) = 5.7%
  95% CI: [68%, 92%]
  
  If CI doesn't include 100% → likely flaky
```

**Interpretation:**
- Pass rate 99.5-100%: Healthy
- Pass rate 95-99%: Acceptable (depending on context)
- Pass rate 80-95%: Flaky
- Pass rate <80%: Very flaky (broken test or code)

---

## Flakiness Severity Levels

| Level | Pass Rate | Action | Example |
|-------|-----------|--------|---------|
| **Green** | 99.5-100% | Monitor only | Happy path test (external service up) |
| **Yellow** | 95-99% | Investigate & fix | Email delivery test (SLA 30s, sometimes slow) |
| **Orange** | 80-95% | Fix before CI | Race condition, timing too tight |
| **Red** | <80% | Fix immediately or skip | Broken test, broken feature |

---

## Root Cause Patterns

### Pattern 1: Timing Issues

**Symptom:** Test passes if you add a `time.sleep()`, fails if you remove it

**Diagnosis:**
- Timeout is too tight
- External service is slow sometimes
- System clock is not synchronized

**Example:**
```python
# Flaky:
response = post("/password-reset")
time.sleep(0.5)  # Only 0.5s
email = get_mailbox()
assert email is not None  # Fails 20% of time

# Fixed:
response = post("/password-reset")
start = time.time()
while time.time() - start < 30:  # Wait up to 30s
    email = get_mailbox()
    if email:
        break
    time.sleep(0.5)
assert email is not None  # Now passes 99%
```

**Fix:** Increase timeout, add exponential backoff, use polling instead of sleep

---

### Pattern 2: Race Conditions

**Symptom:** Test passes when run alone, fails when run with other tests

**Diagnosis:**
- Shared state between tests (database, cache, temp files)
- Test cleanup doesn't actually clean
- Tests run in parallel (concurrency bug)

**Example:**
```python
# Flaky (test pollution):
def setup_method(self):
    self.user = create_user("test@example.com")

def test_login_success(self):
    login(self.user.email, "password123")  # Passes

def test_email_uniqueness(self):
    create_user("test@example.com")  # Fails! User already exists (pollution from previous test)

# Fixed:
def setup_method(self):
    self.user_id = uuid4()
    self.user = create_user(f"test-{self.user_id}@example.com")

def teardown_method(self):
    delete_user(self.user.email)  # Clean up after each test

def test_login_success(self):
    login(self.user.email, "password123")  # Passes

def test_email_uniqueness(self):
    create_user(f"test-{uuid4()}@example.com")  # Now different email, passes
```

**Fix:** Isolate tests (unique IDs, proper teardown), run tests sequentially if needed

---

### Pattern 3: External Service Instability

**Symptom:** Test fails when external service is slow or down

**Diagnosis:**
- Email service is slow
- API endpoint is flaky
- Network timeout
- External service doesn't meet expected SLA

**Example:**
```python
# Flaky (assumes instant email):
def test_reset_email_arrives_immediately(self):
    post("/password-reset", {"email": "test@example.com"})
    email = get_mailbox()  # Might not be there yet!
    assert email is not None  # Fails 10% of time

# Fixed (acknowledges SLA):
def test_reset_email_arrives_within_sla(self):
    post("/password-reset", {"email": "test@example.com"})
    start = time.time()
    email = None
    while time.time() - start < 30:  # SLA is 30s
        email = get_mailbox()
        if email:
            break
        time.sleep(1)
    assert email is not None  # Passes 99%+ (email arrives in 1-5s usually)
    elapsed = time.time() - start
    assert elapsed < 30, f"SLA violated: {elapsed}s"
```

**Fix:** Add retry logic, increase timeout to match SLA, mock external service for tests

---

### Pattern 4: Assertion Ambiguity

**Symptom:** Test fails because assertion is too strict or incomplete

**Diagnosis:**
- Expected value doesn't exactly match actual
- Assertion doesn't account for minor variations
- Oracle claim is ambiguous

**Example:**
```python
# Flaky (too strict):
def test_reset_response_time(self):
    start = time.time()
    response = post("/password-reset")
    elapsed = time.time() - start
    assert elapsed < 1.0  # Fails 30% of time (network variance)

# Fixed (realistic):
def test_reset_response_time(self):
    start = time.time()
    response = post("/password-reset")
    elapsed = time.time() - start
    assert elapsed < 5.0  # SLA is 5 seconds (reasonable for network)

# Or fixed (more granular):
def test_reset_response_time_reasonable(self):
    times = []
    for _ in range(10):
        start = time.time()
        post("/password-reset")
        times.append(time.time() - start)
    
    median = sorted(times)[5]
    p95 = sorted(times)[9]
    
    assert median < 0.5, f"Median {median}s should be <0.5s"
    assert p95 < 2.0, f"P95 {p95}s should be <2s"
```

**Fix:** Loosen assertion, add multiple runs to catch median/percentile, clarify oracle

---

## Measurement Framework

When you detect a flaky test, measure:

```yaml
Test: test_reset_email_arrives_within_30_seconds

Flakiness Score:
  Pass rate: 80% (8/10 runs)
  Severity: Orange (needs fix)
  Consistency: Intermittent (not always same failure)

Environment:
  OS: Linux (CI)
  Python: 3.9
  Email service: staging (SLA 30s)
  Last change: 3 days ago

Failure Pattern:
  Mode: Timeout (email didn't arrive within timeout)
  Frequency: ~20% of runs (2/10)
  Reproducibility: Low (hard to reproduce locally)

Suspected Root Cause:
  Email service in staging is slow sometimes
  Test timeout (1s) is too tight for SLA (30s)
  
Recommended Fix:
  Increase polling timeout from 1s to 30s
  Add exponential backoff: start at 0.1s, max 5s
  Verify: re-run 20 times, expect 100% pass

Effort: 5 minutes (code change)
Impact: Medium (affects email feature, user-facing)
Priority: Yellow (not blocking, but should fix soon)
```

---

## Monitoring Over Time

Track flakiness trends:

```
Test: test_reset_email_arrives_within_30_seconds

Date       Pass Rate  Status    Action
──────────────────────────────────────────
2025-06-01 100%       ✓ Healthy Monitor
2025-06-08 95%        ⚠️ Flaky  Investigate
2025-06-15 80%        ✗ Flaky  Fix (urgent)
2025-06-22 99%        ⚠️ Fixed  (needs verification)
2025-06-29 100%       ✓ Healthy Done
```

**Action Thresholds:**
- New flakiness (was 100%, now <99%): Investigate
- Worsening trend (90% → 80% → 70%): Urgent fix
- Consistent flakiness (stays at 95%): Acceptable, but document

---

## Automation: Flaky Test Detector

Pseudocode for automated detection:

```python
def detect_flaky_tests(test_results):
    """Analyze test results, flag flaky tests"""
    
    flaky_tests = {}
    
    for test_name, results in test_results.items():
        pass_count = sum(1 for r in results if r == "PASS")
        pass_rate = pass_count / len(results)
        
        if pass_rate < 0.99:  # Less than 99%
            flaky_tests[test_name] = {
                "pass_rate": pass_rate,
                "severity": classify_severity(pass_rate),
                "sample_size": len(results),
                "recommendation": recommend_fix(test_name, results)
            }
    
    return flaky_tests

def classify_severity(pass_rate):
    if pass_rate >= 0.995:
        return "Green"
    elif pass_rate >= 0.95:
        return "Yellow"
    elif pass_rate >= 0.80:
        return "Orange"
    else:
        return "Red"
```

---

## Integration with Phase 3 & 4

**Phase 3 (Execute):** If pass rate is <99%, flag for Phase 4

**Phase 4 (Heal):** 
1. Measure flakiness (run 10 times)
2. Diagnose root cause (timing? race condition? external service?)
3. Recommend fix (loosen timeout? improve polling? mock service?)
4. Apply fix
5. Verify (re-run 10 times, expect 100%)

---

## Links

- **Patterns & Fixes:** [self-healing-patterns.md](self-healing-patterns.md)
- **Heal Prompt:** [/prompts/heal/diagnose-test-failure.md](../prompts/heal/diagnose-test-failure.md)
- **Allure Integration:** [/reporting/allure-integration.md](../reporting/allure-integration.md)

---

*Phase 4: Flaky detection and measurement. Know your signal quality.*


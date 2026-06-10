# Self-Healing Patterns

Reusable patterns for fixing common flaky test problems.

---

## Pattern 1: Polling with Backoff

**Problem:** Test polls for a result too fast, misses it or times out

**Symptoms:**
- Email delivery test: "Email not received within timeout"
- Async operation: "Status still pending"
- External service: "Timeout waiting for response"

**Solution:**

```python
import time

def poll_with_backoff(check_func, max_duration=30, initial_wait=0.1, max_wait=5):
    """
    Poll for a condition with exponential backoff.
    
    Args:
        check_func: Function that returns result or None
        max_duration: Max time to wait (seconds)
        initial_wait: Initial wait between polls (seconds)
        max_wait: Max wait between polls (seconds)
    
    Returns:
        Result from check_func, or None if timeout
    """
    start = time.time()
    wait = initial_wait
    
    while time.time() - start < max_duration:
        result = check_func()
        if result is not None:
            return result
        
        time.sleep(wait)
        wait = min(wait * 1.5, max_wait)  # Exponential backoff
    
    return None

# Usage:
def test_reset_email_arrives():
    post("/password-reset", {"email": "test@example.com"})
    
    email = poll_with_backoff(
        check_func=lambda: get_mailbox("test@example.com"),
        max_duration=30,
        initial_wait=0.1,
        max_wait=5
    )
    
    assert email is not None, "Email not received"
```

**Benefits:**
- Starts fast (0.1s wait)
- Backs off if service is slow (max 5s wait)
- Respects SLA (30s timeout)
- Pass rate: 99%+ (catches emails from <1s to 30s)

---

## Pattern 2: Retry with Jitter

**Problem:** Test fails intermittently, but would pass on retry

**Symptoms:**
- Network timeout: "Request timed out"
- Flaky API: "Server returned 503"
- Database locked: "Lock timeout"

**Solution:**

```python
import time
import random

def retry_with_jitter(func, max_attempts=3, base_delay=0.1):
    """
    Retry a function with exponential backoff + jitter.
    
    Args:
        func: Function to retry
        max_attempts: Maximum number of attempts
        base_delay: Initial delay (seconds)
    
    Returns:
        Result, or raises exception if all attempts fail
    """
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts:
                raise
            
            # Exponential backoff with random jitter
            delay = base_delay * (2 ** (attempt - 1))
            jitter = random.uniform(0, delay * 0.1)
            wait = delay + jitter
            
            print(f"Attempt {attempt} failed: {e}. Retrying in {wait:.2f}s...")
            time.sleep(wait)

# Usage:
def test_login_with_retries():
    def attempt_login():
        response = post("/login", {"email": "test@example.com", "password": "pass"})
        if response.status_code >= 500:
            raise Exception(f"Server error: {response.status_code}")
        return response
    
    response = retry_with_jitter(attempt_login, max_attempts=3)
    assert response.status_code == 200
```

**Benefits:**
- Handles transient failures (network blips, server hiccups)
- Jitter prevents thundering herd (all retries at same time)
- Pass rate: 95%+ (catches most transient issues)

---

## Pattern 3: Test Isolation with Random IDs

**Problem:** Tests pollute each other; shared state causes failures

**Symptoms:**
- "User already exists" (another test created same user)
- "Email already registered" (conflict with parallel test)
- Database state from previous test

**Solution:**

```python
import uuid

class TestPasswordReset:
    """Isolated tests with unique IDs"""
    
    def setup_method(self):
        """Create unique test user for each test"""
        self.user_id = str(uuid.uuid4())[:8]
        self.email = f"test-{self.user_id}@example.com"
        self.password = "TestPass123!"
        
        # Create user
        create_user(email=self.email, password=self.password)
    
    def teardown_method(self):
        """Clean up test user"""
        delete_user(email=self.email)
    
    def test_user_can_reset_password(self):
        # Each test has unique email, no conflicts
        response = post("/password-reset", {"email": self.email})
        assert response.status_code == 200
    
    def test_old_password_doesn't_work(self):
        # Reset password
        reset_password(self.email, "NewPassword123!")
        
        # Old password shouldn't work
        response = post("/login", {
            "email": self.email,
            "password": self.password  # Old password
        })
        assert response.status_code == 401
```

**Benefits:**
- No test pollution (each test has unique user)
- Safe to run in parallel
- Cleanup guaranteed (teardown always runs)
- Pass rate: 100% (isolated tests are reliable)

---

## Pattern 4: Loose Assertions with Tolerances

**Problem:** Assertion is too strict; small variations cause failures

**Symptoms:**
- "Expected 1.0, got 1.001" (timing variation)
- "Expected X, got Y" (whitespace, minor formatting)
- "Expected exactly 100, got 101" (off-by-one due to clock skew)

**Solution:**

```python
def test_response_time_reasonable():
    """Assert response time is reasonable (not exact)"""
    start = time.time()
    response = post("/password-reset")
    elapsed = time.time() - start
    
    # Loose assertion: response should be reasonably fast
    assert elapsed < 5.0, f"Response too slow: {elapsed}s (SLA: 5s)"
    
    # Or: assert median over multiple runs
    times = [measure_response_time() for _ in range(5)]
    median = sorted(times)[2]
    assert median < 1.0, f"Median response time {median}s (expected <1s)"


def test_email_message_contains_expected_text():
    """Assert email contains expected text (not exact match)"""
    email = get_mailbox()
    
    # Loose: just check key parts are there
    assert "reset" in email.subject.lower()
    assert "http" in email.body  # URL is present
    assert "token=" in email.body  # Token is in URL
    
    # NOT: assert email.subject == "Exactly this"


def test_token_expires_approximately_24_hours():
    """Token expires around 24 hours (±5 minutes tolerance)"""
    request_time = datetime.now()
    
    # Create token
    token = create_reset_token()
    
    # Try to use it after 24h + 0.5h (beyond 24h, should fail)
    response = use_token_after_delay(token, delay_seconds=24*3600 + 30*60)
    assert response.status_code == 401, "Token should expire after ~24h"
    
    # Try to use it after 23.5h (before 24h, should work)
    response = use_token_after_delay(token, delay_seconds=23.5*3600)
    assert response.status_code == 200, "Token should be valid before 24h"
```

**Benefits:**
- Accounts for system variance (network, clock skew, etc.)
- Assertions still catch real bugs (not so loose they're useless)
- Pass rate: 99%+ (tolerances are realistic)

---

## Pattern 5: Conditional Assertions (Accept Multiple Valid States)

**Problem:** Test assumes one behavior, but code implements equivalent alternative

**Symptoms:**
- "Expected HTTP 200, got 201" (both indicate success, different conventions)
- "Expected 'success', got 'OK'" (different message, same meaning)
- "Expected property x, got property y" (different structure, same information)

**Solution:**

```python
def test_reset_request_accepted():
    """Password reset request is accepted (HTTP 200 or 202)"""
    response = post("/password-reset", {"email": "test@example.com"})
    
    # Accept either 200 (immediate) or 202 (accepted, processing)
    assert response.status_code in [200, 202], \
        f"Expected 200/202, got {response.status_code}"


def test_reset_confirmation_message_present():
    """Confirmation message is present (various valid formats)"""
    response = post("/password-reset", {"email": "test@example.com"})
    data = response.json()
    
    # Accept any of these messages
    message = data.get("message", "")
    valid_messages = [
        "reset link has been sent",
        "if that email is registered",
        "check your email",
        "confirmation sent"
    ]
    
    assert any(m in message.lower() for m in valid_messages), \
        f"Message doesn't contain expected text: {message}"


def test_reset_token_in_response_or_email():
    """Token is either in response OR in email (either is valid)"""
    response = post("/password-reset", {"email": "test@example.com"})
    
    # Try to find token in response first
    token = response.json().get("token")
    
    # If not in response, must be in email
    if token is None:
        email = get_mailbox()
        assert "token=" in email.body, "Token must be in response or email"
    else:
        assert token, "Token should be non-empty"
```

**Benefits:**
- Tests implementation flexibility (different ways to achieve same goal)
- Doesn't fail on semantically equivalent responses
- Pass rate: 99%+ (accepts valid variations)

---

## Pattern 6: Feature Flag / Skip Flaky Tests (Temporary)

**Problem:** You've diagnosed a flaky test but can't fix it immediately

**Solution:** Skip with reason, plan fix

```python
import pytest

@pytest.mark.skip(reason="Flaky: email service timeout (SLA 30s, test timeout 1s). "
                         "Fix: increase timeout to 30s. "
                         "Ticket: ISSUE-123 "
                         "Estimated fix: 2025-06-15")
def test_reset_email_arrives_immediately():
    """TODO: This test is flaky, needs timeout increase"""
    response = post("/password-reset")
    email = get_mailbox()
    assert email is not None  # Too fast, sometimes fails


# Or: Flaky but don't skip (for monitoring):

@pytest.mark.flaky(reruns=3, reruns_delay=2)  # pytest-rerunfailures plugin
def test_reset_email_arrives():
    """This test is flaky; re-run up to 3 times"""
    response = post("/password-reset")
    time.sleep(0.5)
    email = get_mailbox()
    assert email is not None
```

**Benefits:**
- Documents why test is skipped
- Includes fix plan with deadline
- Doesn't hide flakiness (still visible in test suite)
- Can be tracked in metrics

**Use only as temporary measure while fix is implemented.**

---

## Healing Checklist

When you encounter a flaky test:

- [ ] **Measure:** Run 10 times, calculate pass rate
- [ ] **Diagnose:** Identify root cause (timing? race? external service?)
- [ ] **Classify:** Is this a test bug or product bug?
- [ ] **Select pattern:** Which fix pattern applies?
- [ ] **Implement:** Apply the pattern
- [ ] **Verify:** Re-run 10 times, expect 100% pass
- [ ] **Commit:** Document why it was flaky, what fixed it
- [ ] **Monitor:** Track if it re-appears

---

## Integration with Phase 4

1. **Phase 3 (Execute):** Detects test failures
2. **Phase 4 (Heal):** Uses these patterns to diagnose and fix
3. **Self-Healing:** Automatically applies recommended fixes
4. **Phase 5 (Analyze):** Reassesses readiness

---

## Links

- **Flaky Detection:** [flaky-test-detection.md](flaky-test-detection.md)
- **Heal Prompt:** [/prompts/heal/diagnose-test-failure.md](../prompts/heal/diagnose-test-failure.md)

---

*Phase 4: Self-healing patterns. Fix flakiness systematically.*


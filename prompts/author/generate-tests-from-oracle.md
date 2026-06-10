# Generate Tests From Oracle

**Version:** 0.1  
**Stage:** Author  
**Grounded in:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md)

---

## Overview

Convert an oracle (checklist of acceptance criteria and edge cases) into test code.

Each claim in the oracle becomes one or more test functions. The test names should be readable—when you see the test name, you should see the oracle claim.

---

## Prerequisites

- An oracle (from `/prompts/plan/extract-oracle-from-prd.md`)
- A target language and test framework (Python + pytest, JavaScript + Jest, Java + TestNG, etc.)
- Optionally: sample test code in that language (to match style/conventions)
- Optionally: API/UI documentation (to know which endpoints or UI elements to test)

---

## The Prompt

```
You are a test author. Your job is to write test code that directly maps to 
an oracle (a checklist of acceptance criteria).

Each claim in the oracle becomes one test function. The test name should be 
crystal clear—reading the test name, a human should see exactly what's being 
verified.

DO NOT:
- Over-engineer (no page objects, no helper libraries, no frameworks beyond what's needed)
- Test things not in the oracle
- Make assumptions about implementation
- Write complex setup/teardown logic yet (we'll scaffold that next)

DO:
- Write one test per claim (or one per closely related group)
- Use clear, descriptive test names
- Assert exactly what the oracle says, not more
- Include a comment (one line) explaining the oracle claim being tested
- Make tests runnable (but they may need setup/fixtures we'll scaffold separately)

---

## The Oracle:
[INSERT ORACLE HERE]

---

## Target Language/Framework:
[e.g., "Python with pytest"]

---

## Your Task:

For each claim in the oracle (Happy Path, Edge Cases, Error Conditions, Security):
1. Write a test function with a clear, readable name
2. Add a one-line comment explaining the oracle claim
3. Implement the test logic
4. Include assertions that match the oracle claim exactly

---

## Example Format (Pytest):

```python
def test_user_can_request_password_reset():
    """Oracle: User can request a password reset by entering their email."""
    response = post("/password-reset", {"email": "user@example.com"})
    assert response.status_code == 200
    assert "reset link has been sent" in response.json()["message"]


def test_reset_email_arrives_within_30_seconds():
    """Oracle: System sends a password reset email within 30 seconds."""
    start = time.time()
    post("/password-reset", {"email": "user@example.com"})
    email = poll_mailbox("user@example.com", timeout=30)
    elapsed = time.time() - start
    assert email is not None, "Email did not arrive"
    assert elapsed <= 30, f"Email took {elapsed}s (expected ≤30s)"


def test_reset_link_expires_after_24_hours():
    """Oracle: Reset link is no longer valid after 24 hours."""
    response = post("/password-reset", {"email": "user@example.com"})
    token = extract_token_from_email(response)
    # Simulate 25 hours passing (or use time-travel in tests)
    response = post("/reset", {"token": token, "new_password": "NewPass123"})
    assert response.status_code == 401, "Expected token to be expired"


def test_user_cannot_use_old_password_after_reset():
    """Oracle: User cannot log in with the old password after reset."""
    # Reset password to new_password
    reset_password("user@example.com", "NewPassword123")
    # Try to log in with old password
    response = post("/login", {"email": "user@example.com", "password": "OldPassword"})
    assert response.status_code == 401, "Old password should not work"


def test_invalid_email_returns_error():
    """Oracle: Invalid email address results in an error message."""
    response = post("/password-reset", {"email": "not-an-email"})
    assert response.status_code == 400
    assert "invalid email" in response.json()["error"].lower()


def test_nonexistent_email_returns_generic_error():
    """Oracle: Nonexistent email does not reveal whether email exists (security)."""
    response = post("/password-reset", {"email": "doesnotexist@example.com"})
    # Should NOT say "email not found"; should be generic
    assert response.status_code == 200  # Accepted (not 404)
    assert "reset link has been sent" in response.json()["message"]  # Lie for security


def test_expired_reset_link_returns_clear_error():
    """Oracle: Expired reset link results in a clear error message."""
    token = create_expired_token()
    response = post("/reset", {"token": token, "new_password": "NewPass123"})
    assert response.status_code == 401
    assert "expired" in response.json()["error"].lower()
    assert "request a new link" in response.json()["error"].lower()
```

---

## Output Format:

Provide the test code as a single runnable file (or multiple if the language requires). 
Each test should:
- Have a descriptive name (test_* in pytest, test* in Jest, etc.)
- Include a one-line docstring/comment referencing the oracle claim
- Be self-contained (or note what fixtures/setup are needed)
- Include assertions that match the oracle exactly

If fixtures are needed (e.g., database setup, test user creation), call them out in a 
comment like:
```
# Fixture needed: authenticated_user, test_email_account
```

---

## Tradeoffs

| Aspect | Tradeoff | Note |
|--------|----------|------|
| **Effort** | 10-30 min per oracle | Depends on oracle size and test framework familiarity. |
| **Code quality** | ~60% baseline (needs review) | LLM may miss edge cases, write flaky tests, or over-complicate. Always review. |
| **Framework assumptions** | Minimal | Uses only stdlib + framework basics. No custom DSLs. |
| **Runability** | 70% first-time pass | Tests compile/parse but may fail due to missing fixtures or environment setup. |
| **Maintainability** | Good | Names are clear; structure mirrors oracle. Easy to refactor. |

---

## When to Use This Prompt

- **You have:** A clear oracle (checklist)
- **You want:** Test code that directly maps to the oracle
- **You're building:** A new test suite or adding tests to an existing one

---

## When NOT to Use This Prompt

- The oracle is vague or incomplete (refine the oracle first)
- You're fixing an existing test suite (use `/prompts/heal/` instead)
- You need load testing, performance testing, or security scanning (use specialized tools/prompts)

---

## Next Steps

1. **Review the tests** — do they match the oracle? Are the names clear?
2. **Set up fixtures** — add test data, authentication, database setup
3. **Run them** — see which ones pass, which ones fail, which ones are flaky
4. **Refine** — fix flaky tests, clarify oracle if tests revealed gaps

Once tests are ready to run:
- Feed them to `/prompts/execute/run-tests-and-report.md` to run and collect results

---

## Links

- **Concept:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md) — why oracles matter
- **Prior stage:** [extract-oracle-from-prd.md](../plan/extract-oracle-from-prd.md)
- **Next stage:** [run-tests-and-report.md](../execute/run-tests-and-report.md)

---

*Last updated: Phase 1 v0.1*

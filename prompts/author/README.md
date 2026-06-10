# Author: Oracle → Test Code

Given an oracle (checklist of acceptance criteria), generate test code: test functions, data factories, parametrization, helper utilities.

## What Goes Here

Prompts for:
- Generating test functions from a checklist (happy path, edge cases, error cases)
- Creating data factories and fixtures
- Writing parametrized tests (same logic, multiple inputs)
- Scaffolding test layers (page objects for UI, API clients for APIs)
- Contract tests (schema validation, request/response format)
- Setting up test data (given/when/then)

## Input

An oracle checklist (from `/plan`), a target language/framework (Pytest, Jest, TestNG, etc.), and optionally example tests.

## Output Goal

**Test code** that directly maps to the oracle. Ideally, you can read the test names and see the checklist covered. No over-engineering, no tests for things not in the oracle.

## Example Use

**Input Oracle:**
```
- User can request password reset
- Reset email arrives within 30s
- Reset link expires after 24h
- User cannot reuse old password
```

**Output (Pytest):**
```python
def test_user_can_request_password_reset():
    # POST /password-reset with valid email
    # Expect: 202 Accepted, confirmation message

def test_reset_email_arrives_within_30s():
    # Request reset, poll /reset-status for delivery
    # Expect: email arrived, timestamp < 30s

def test_reset_link_expires_after_24h():
    # Request reset, extract token, wait 24h+, use token
    # Expect: 401 Unauthorized, token expired

def test_user_cannot_reuse_old_password():
    # Reset with new password, attempt login with old
    # Expect: 401 Unauthorized
```

Each test name is a claim from the oracle. The executor runs them.

---

*Last updated: Phase 0 — skeleton created*


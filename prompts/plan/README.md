# Plan: Requirement → Oracle

Convert a requirement (PRD, spec, user story) into a **testable oracle**: acceptance criteria, test scope, edge cases, and risk ranking.

## What Goes Here

Prompts for:
- Summarizing requirements and extracting testable claims
- Identifying edge cases (boundary values, error conditions, security cases)
- Creating explicit acceptance criteria (Given/When/Then format)
- Risk-ranking test cases (critical, high, medium, low)
- Asking clarifying questions when a requirement is ambiguous
- Generating a contract/API spec from documentation

## Output Goal

A **checklist** — structured, explicit, prioritized — that the test author and executor can work from. This is the oracle.

## Example Use

**Input:** A PRD that says "Users can reset their password via email."

**Output:**
```
# Password Reset Oracle

## Happy Path (Critical)
- User can request password reset
- Reset email arrives within 30s
- Reset link in email is valid for 24h
- User can set new password via link
- Session is logged out after reset

## Edge Cases (High)
- Reset link expires after 24h
- User cannot use old password after reset
- Multiple reset requests in quick succession
- Reset fails gracefully with offline SMTP

## Security (Critical)
- Reset token is cryptographically secure
- Token is not leaked in logs, error messages, or referrers
- User identity is verified (not enumerable)

## UI/UX (Medium)
- Clear success message after reset
- Clear error message if token expired
```

The tester now has an explicit oracle. The agent can run tests against it.

---

*Last updated: Phase 0 — skeleton created*


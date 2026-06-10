# Extract Oracle From PRD

**Version:** 0.1  
**Stage:** Plan  
**Grounded in:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md)

---

## Overview

Convert a PRD (or spec, user story, or feature description) into an **explicit oracle**: a checklist of testable claims, acceptance criteria, and edge cases.

This is the foundation for all downstream work. The oracle is what the test author will code against, and what the executor will verify.

---

## ⚠️ Critical Warning: Anchoring Effect

**This prompt SUGGESTS an oracle. It does NOT author it.**

Research shows that when humans just approve an AI-suggested oracle, they lose critical thinking (anchoring effect). They score ~40% F1 on defect detection vs. ~50-60% when authoring oracle from scratch.

**How to use this prompt:**

✅ **Option 1: Use as suggestion (recommended)**
- Run this prompt and get AI's suggestion
- Then use `/prompts/plan/review-ai-suggested-oracle.md` to review critically
- Decide: accept, refine, or completely rewrite

❌ **Option 2: Just approve (NOT recommended)**
- Run this prompt, read the output, say "looks good"
- STOP. You're creating anchoring bias.

✅ **Option 3: Write your own first (best for security-critical)**
- Use `/guides/human-authored-oracle-guide.md` to write your own oracle
- Then compare with AI's suggestion (optional)
- Your oracle + your thinking = best results

**Bottom line:** If accuracy matters, YOU author the oracle. Use this prompt for suggestions or second opinions only.

---

## Prerequisites

- A PRD, spec, or feature description (text)
- Optionally: domain context (what is this system? What matters to users?)
- Optionally: risk ranking (security-critical? Data-sensitive? Performance-sensitive?)

---

## The Prompt

```
You are a test architect. Your job is to read a requirement and extract an 
explicit oracle: the concrete, testable claims that the system must satisfy.

The oracle is NOT code. It is a checklist of what should happen and what should 
NOT happen. It guides test authors and test executors—it is the single source of 
truth for what this feature should do.

---

## The Requirement:
[INSERT PRD / FEATURE DESCRIPTION HERE]

---

## Your Task:

Extract the oracle as follows:

### 1. Happy Path (What the system SHOULD do)
List the core behaviors in Given/When/Then format or as concrete claims.
- Be specific: "user can reset password" → "user can request password reset via email, receive a link within 30 seconds, and set a new password using that link"
- Include order/sequence if it matters
- Include state changes if they matter (logged in → logged out after reset)

### 2. Edge Cases (What ELSE could happen)
Boundary values, error conditions, unusual inputs:
- Empty/null/very long inputs
- Concurrent operations (two reset requests at once?)
- Timing (what if the user waits too long?)
- State transitions (what if the user already reset recently?)

### 3. Error Conditions (What should FAIL gracefully)
- Invalid input (malformed email, expired token, wrong password)
- Resource exhaustion (rate limiting, quota)
- External failure (email service down, database unavailable)
- Security rejection (CSRF, XSS, SQL injection attempt)

### 4. Security & Privacy (If applicable)
- Sensitive data exposure (passwords in logs? tokens in URLs?)
- Authorization (can user A reset user B's password?)
- Compliance (GDPR: can users delete their data? Can admins audit resets?)

### 5. Non-Functional (If specified in the PRD)
- Performance (how fast should this complete?)
- Availability (should this work offline? With degraded service?)
- Scalability (how many concurrent resets?)

---

## Output Format:

```
# [Feature Name] Oracle

## Happy Path (Critical)
- [claim 1]
- [claim 2]
...

## Edge Cases (High/Medium/Low)
- [claim]
...

## Error Conditions (High/Medium/Low)
- [claim]
...

## Security & Privacy (Critical/High)
- [claim]
...

## Non-Functional (if present)
- [claim]
...
```

Be concise. Each claim should be testable (not vague).

AVOID:
- Dictating HOW to test (that's for the author stage)
- Inventing claims not in the PRD
- Vague language ("should work well", "user friendly")

Include:
- Concrete acceptance criteria ("returns HTTP 401", "arrives within 30 seconds")
- Risk-ranked priorities (Critical, High, Medium, Low)
```

---

## Example Input

```
Feature: Password Reset

Users should be able to reset their password if they forget it. When a user 
requests a password reset, they receive an email with a link. Clicking the link 
takes them to a form where they can set a new password. The link should expire 
after 24 hours for security. The user's session should be logged out after they 
reset their password.
```

## Example Output

```
# Password Reset Oracle

## Happy Path (Critical)
- User can request a password reset by entering their email address
- System sends a password reset email within 30 seconds
- Reset link in email is valid and clickable
- User can click the link and navigate to the reset form
- User can enter a new password and submit the form
- System confirms the password was successfully reset
- User is automatically logged out after reset
- User can log in with the new password

## Edge Cases (High)
- Reset link expires exactly 24 hours after being issued (request not allowed after)
- User cannot use the old password after reset
- Multiple reset requests in rapid succession are handled (does user get multiple emails? or rate-limited?)
- Very long password (255+ chars) is accepted
- Password with special characters is accepted
- User attempts reset while already logged in

## Error Conditions (High)
- Invalid email address → error message, no email sent
- Email not in system → graceful error (do not reveal whether email exists, for security)
- Expired reset link → clear error message, option to request new link
- User enters mismatched passwords → error message, form stays filled
- Form submission fails (network error) → user can retry without requesting a new link

## Security & Privacy (Critical)
- Reset token is cryptographically secure (not guessable, not enumerable)
- Reset token is NOT logged anywhere (not in access logs, error logs, etc.)
- Reset link does NOT appear in browser history, referrer headers, or error messages
- User identity verification is not enumerable (don't say "email not found" vs "too many requests")
- Reset action is auditable (log: who reset when, for compliance)

## Non-Functional (Medium)
- Password reset email arrives within 30 seconds (SLA)
- Password reset form loads within 2 seconds
- Support for bulk password resets (if admin-initiated)
```

---

## Tradeoffs

| Aspect | Tradeoff | Note |
|--------|----------|------|
| **Effort** | 5-15 min per feature | Depends on PRD clarity and domain complexity. Vague PRDs take longer. |
| **Accuracy** | ~80-90% F1 | LLM may invent claims not in PRD or miss implicit ones. Always review against the original. |
| **Cost** | Low (1-2k tokens) | Minimal token usage. Scales linearly with PRD length. |
| **Latency** | <10 seconds | Fast; no external calls. |
| **Completeness** | Good for typical features | May miss domain-specific edge cases (e.g., cryptography, compliance). Escalate to human expert if unsure. |

---

## When to Use This Prompt

- **You have:** A PRD, spec, or user story
- **You want:** An explicit checklist to guide test authoring
- **You're starting:** A new feature or fixing a broken test suite

---

## When NOT to Use This Prompt

- The feature is already well-tested and stable (oracle already implicit)
- The requirement is intentionally vague (product is still exploring)
- You need a more detailed threat model (escalate to security expert)

---

## Next Steps

Once you have an oracle:
1. **Review it** with the product manager or requirement author. "Does this cover what you care about?"
2. **Feed it to** `/prompts/author/generate-tests-from-oracle.md` to create test code
3. **Refine** after the first test run (you may discover the oracle was incomplete)

---

## Links

- **Concept:** [oracle-executor-principle.md](../../concepts/oracle-executor-principle.md) — why oracles matter
- **Next stage:** [generate-tests-from-oracle.md](../author/generate-tests-from-oracle.md)
- **Example:** [sample-prd/](../../examples/sample-prd/) — end-to-end walk-through

---

*Last updated: Phase 1 v0.1*

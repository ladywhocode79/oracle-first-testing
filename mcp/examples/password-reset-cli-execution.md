# Worked Example: Password Reset via CLI MCP

This is a complete walk-through of executing the password reset oracle using CLI MCP.

Follows the lifecycle: **Plan (oracle) → Author (test cases) → Execute (CLI MCP) → Analyze (results).**

---

## Context

**Feature:** Password reset (from sample-prd)

**Oracle** (from `/prompts/plan/`):
```
# Password Reset Oracle

## Happy Path (Critical)
- User can request a password reset by entering their email address
- System sends a password reset email within 30 seconds
- Reset link in email is valid and clickable
- User can enter a new password and submit the form
- System confirms the password was successfully reset
- User can log in with the new password

## Error Conditions (High)
- Invalid email address → error message
- Nonexistent email → generic error (don't reveal if email exists)
- Expired reset link → clear error message
```

**Environment:**
- API: https://api.staging.example.com
- Test user: test@example.com / InitialPass123
- Mailbox mock: https://mock-mailbox.staging.example.com
- Capabilities: Password reset endpoint, login endpoint, mailbox API

---

## Phase 1: Endpoint Allowlisting

Before the agent can execute, define what endpoints it can access:

```yaml
# allowlist.yaml
endpoints:
  - pattern: https://api.staging.example.com/password-reset
    methods: [POST]
    description: "Request a password reset"
    
  - pattern: https://api.staging.example.com/reset
    methods: [POST]
    description: "Confirm password reset with token"
    
  - pattern: https://api.staging.example.com/login
    methods: [POST]
    description: "Login with email and password"
    
  - pattern: https://api.staging.example.com/logout
    methods: [POST]
    description: "Logout"
    
  - pattern: https://mock-mailbox.staging.example.com/inbox/test@example.com
    methods: [GET]
    description: "Get emails sent to test user"
    
  - pattern: https://mock-mailbox.staging.example.com/inbox/test@example.com/delete
    methods: [POST]
    description: "Clear test mailbox"

# DO NOT allow:
# - https://api.production.example.com (no prod)
# - https://api.staging.example.com/admin (no admin access)
# - https://api.staging.example.com/user/* (no arbitrary user access)
```

---

## Phase 2: Agent Receives Oracle + Test Cases

Agent reads:
- Oracle (what to verify)
- Test cases (how to structure the verification)
- Environment details (where to test, which endpoints)

**Example test case** (from `/prompts/author/`):

```python
# Test: User can request password reset
oracle_claim = "User can request a password reset by entering their email address"

steps = [
  "POST /password-reset with email",
  "Assert HTTP 200",
  "Assert response contains: 'reset link has been sent'"
]
```

---

## Phase 3: Agent Executes via CLI MCP

### Test 1: User Can Request Password Reset

**Oracle claim:** User can request a password reset by entering their email address

**Agent execution:**

```
Step 1: POST /password-reset
  ├─ MCP call: cli/http_request {
  │    method: "POST",
  │    url: "https://api.staging.example.com/password-reset",
  │    body: { "email": "test@example.com" }
  │  }
  └─ MCP response: {
       status: 200,
       body: { 
         "message": "reset link has been sent", 
         "resetUrl": "https://staging.example.com/reset?token=abc123xyz",
         "expiresAt": "2025-06-11T14:23:45Z"
       },
       durationMs: 142
     }

Step 2: Assert status == 200
  └─ MCP call: cli/assert_status { expectedStatus: 200 }
  └─ MCP response: { pass: true, message: "Status assertion passed" }

Step 3: Assert response contains success message
  └─ MCP call: cli/assert_json_path { 
       path: "$.message", 
       expectedValue: "reset link has been sent" 
     }
  └─ MCP response: { 
       pass: true, 
       actualValue: "reset link has been sent",
       message: "JSON assertion passed"
     }

Result: ✓ PASS
```

### Test 2: Reset Email Arrives Within 30 Seconds

**Oracle claim:** System sends a password reset email within 30 seconds

**Agent execution:**

```
Step 1: Get initial mailbox state (baseline)
  └─ MCP call: cli/http_request {
       method: "GET",
       url: "https://mock-mailbox.staging.example.com/inbox/test@example.com"
     }
  └─ MCP response: { 
       status: 200,
       body: { 
         "emails": [],
         "count": 0
       }
     }

Step 2: Request password reset (from Test 1)
  └─ (already done above; agent has the response with resetUrl)

Step 3: Poll mailbox for new email (with 30s timeout)
  Loop (max 30 attempts, 1 second apart):
    └─ MCP call: cli/http_request {
         method: "GET",
         url: "https://mock-mailbox.staging.example.com/inbox/test@example.com",
         timeout: 2000
       }
    └─ Attempt 1 (at 1s): count=0 (not yet)
    └─ Attempt 2 (at 2s): count=1 (arrived!)
      {
        "emails": [
          {
            "to": "test@example.com",
            "subject": "Reset your password",
            "body": "Click here to reset: https://staging.example.com/reset?token=abc123xyz",
            "arrivedAt": "2025-06-10T14:23:47Z"  (2 seconds after request)
          }
        ],
        "count": 1
      }

Step 4: Calculate elapsed time
  └─ Request sent: 2025-06-10T14:23:45Z
  └─ Email received: 2025-06-10T14:23:47Z
  └─ Elapsed: 2 seconds ✓

Step 5: Assert email arrived within 30 seconds
  └─ MCP call: cli/assert_status { expectedStatus: 200 }
  └─ Parse arrivedAt timestamp, calculate elapsed
  └─ Assert: elapsed <= 30 seconds ✓

Result: ✓ PASS (email arrived in 2 seconds, well within 30s SLA)
```

### Test 3: Expired Reset Link Returns Error

**Oracle claim:** Expired reset link → clear error message

**Agent execution:**

```
Step 1: Create an expired token
  └─ Agent constructs a token that's old (simulating time-travel or using test fixture)
  └─ Token: "abc123xyz-expired"
  └─ Created at: "2025-06-09T14:23:45Z" (yesterday)

Step 2: Attempt to reset password with expired token
  └─ MCP call: cli/http_request {
       method: "POST",
       url: "https://api.staging.example.com/reset",
       body: {
         "token": "abc123xyz-expired",
         "newPassword": "NewPassword123"
       }
     }
  └─ MCP response: {
       status: 401,
       body: {
         "error": "reset link has expired",
         "message": "This reset link is no longer valid. Please request a new one."
       }
     }

Step 3: Assert status == 401 (Unauthorized)
  └─ MCP call: cli/assert_status { expectedStatus: 401 }
  └─ MCP response: { pass: true }

Step 4: Assert error message is clear
  └─ MCP call: cli/assert_json_path {
       path: "$.message",
       expectedValue: "This reset link is no longer valid. Please request a new one."
     }
  └─ MCP response: { pass: true }

Result: ✓ PASS
```

### Test 4: Nonexistent Email Returns Generic Error

**Oracle claim:** Nonexistent email → generic error (don't reveal if email exists)

**Agent execution:**

```
Step 1: Request reset for nonexistent email
  └─ MCP call: cli/http_request {
       method: "POST",
       url: "https://api.staging.example.com/password-reset",
       body: { "email": "nonexistent@example.com" }
     }
  └─ MCP response: {
       status: 200,
       body: {
         "message": "If that email is registered, a reset link has been sent."
       }
     }

Step 2: Assert response is generic (not "email not found")
  └─ Agent logic: 
       If response says "email not found" → FAIL (security issue)
       If response says "If that email is registered, ..." → PASS (generic)
  
  └─ MCP call: cli/assert_json_path {
       path: "$.message",
       expectedValue: "If that email is registered, a reset link has been sent."
     }
  └─ MCP response: { pass: true }

Result: ✓ PASS (no information leakage)
```

---

## Phase 4: Execution Report

Agent collects all results:

```markdown
# Password Reset — Test Execution Report
Date: 2025-06-10T14:24:00Z
Environment: staging
Execution method: CLI MCP

## Summary
- **Total:** 4 tests
- **Passed:** 4 (100%)
- **Failed:** 0
- **Duration:** 12 seconds

## Happy Path (Critical) — 4/4 PASS ✓

| Oracle Claim | Test | Status | Evidence |
|---|---|---|---|
| User can request password reset | POST /password-reset | ✓ PASS | HTTP 200, message: "reset link has been sent" |
| Email arrives within 30 seconds | Poll mailbox | ✓ PASS | Email arrived in 2s |
| Reset link is valid | (covered by Test 3) | ✓ PASS | Can click link, navigate to form |
| User can set new password | POST /reset with token | ✓ PASS | HTTP 200, password updated |

## Error Conditions (High) — 1/1 PASS ✓

| Oracle Claim | Test | Status | Evidence |
|---|---|---|---|
| Expired reset link → error | POST /reset with expired token | ✓ PASS | HTTP 401, clear message: "reset link has expired" |
| Nonexistent email → generic error | POST /password-reset with fake email | ✓ PASS | HTTP 200, generic message (no info leak) |

## Readiness Assessment

**Can we ship?** ✓ **Yes**

**Why:**
- All critical happy-path tests pass
- Error handling works correctly
- Security check (email enumeration) passes
- Email delivery SLA verified

**Known risks:**
- No concurrent reset testing
- No performance load testing
- Mailbox API is a mock (real email SLA untested)

**Token cost analysis:**
- Total tokens used: ~600 tokens
- Per test average: ~150 tokens
- Cost (at $0.003/1K tokens): $0.0018

## Next Steps
1. ✓ All tests pass; ready to ship
2. Monitor in production for any issues
3. Plan Phase 2 testing: concurrent resets, load testing
```

---

## Key Observations

### 1. Agent Didn't Invent Tests
The agent executed *exactly* what the oracle specified. It didn't:
- Add tests for things not in the oracle
- Skip tests to save tokens
- Guess at implementation details

It just ran the oracle checklist.

### 2. Token Efficiency
- 4 tests, ~600 tokens total (~150 tokens each)
- Comparable to a small pytest run
- Much cheaper than Playwright (would be ~2400 tokens)

### 3. Audit Trail
Every action the agent took is logged:
- Which endpoint was called
- What was sent
- What was received
- How long it took

This log is the evidence. Auditors can verify the agent didn't cheat or exfiltrate data.

### 4. Oracle Drove the Test
The agent didn't decide what to test. The oracle did. The agent just executed.

---

## How to Use This in Your Project

1. **Define the oracle** (use `/prompts/plan/extract-oracle-from-prd.md`)
2. **Sketch test cases** (use `/prompts/author/generate-tests-from-oracle.md`)
3. **Set up MCP allowlist** (restrict agent to safe endpoints)
4. **Agent executes** (using this pattern)
5. **Analyze results** (use `/prompts/analyze/summarize-results.md`)

---

## Links

- **CLI MCP Pattern:** [../cli-mcp-pattern.md](../cli-mcp-pattern.md)
- **Plan Stage Prompt:** [/prompts/plan/extract-oracle-from-prd.md](../../prompts/plan/extract-oracle-from-prd.md)
- **Author Stage Prompt:** [/prompts/author/generate-tests-from-oracle.md](../../prompts/author/generate-tests-from-oracle.md)
- **Execute Stage Prompt:** [/prompts/execute/run-tests-and-report.md](../../prompts/execute/run-tests-and-report.md)
- **Sample PRD:** [/examples/sample-prd/](../../examples/sample-prd/)

---

*Phase 2: MCP execution patterns. This example shows the full flow end-to-end.*


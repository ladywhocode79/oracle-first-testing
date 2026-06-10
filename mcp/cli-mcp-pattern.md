# CLI MCP Pattern

A Model Context Protocol server for controlled API and command-line execution.

---

## What It Does

Exposes HTTP requests and shell commands as MCP tools, but with strict allowlisting:

✓ **Allowed:** GET/POST/PUT/DELETE to whitelisted endpoints, with validated payloads
✗ **Blocked:** Arbitrary shell commands, accessing sensitive files, writing to filesystem

This prevents the agent from:
- Running `rm -rf /` or any destructive command
- Reading `/etc/passwd` or other sensitive files
- Accessing endpoints not on the whitelist (prod, admin endpoints, etc.)
- Sending requests with sensitive data uncontrolled

But allows the agent to:
- Make REST API calls like a normal client
- Verify HTTP status codes and response bodies
- Parse JSON responses
- Assert on response structure and values

---

## MCP Tool Definitions

The server exposes these tools:

### HTTP Requests

```
Tool: cli/http_request
Input: {
  method: "GET" | "POST" | "PUT" | "DELETE" | "PATCH",
  url: string,
  headers?: object,
  body?: object | string,
  timeout?: number
}
Output: {
  status: number,
  statusText: string,
  headers: object,
  body: string | object,
  durationMs: number
}

Example:
  Input: {
    method: "POST",
    url: "https://api.staging.example.com/password-reset",
    headers: { "Content-Type": "application/json" },
    body: { email: "test@example.com" }
  }
  Output: {
    status: 200,
    statusText: "OK",
    headers: { "content-type": "application/json" },
    body: { message: "reset link has been sent", resetUrl: "..." },
    durationMs: 145
  }
```

### Assertions

```
Tool: cli/assert_status
Input: { expectedStatus: number }
Output: { pass: boolean, actualStatus: number, message: string }

Tool: cli/assert_json_path
Input: { path: string, expectedValue?: any }
Output: { pass: boolean, actualValue: any, message: string }

Tool: cli/assert_header
Input: { header: string, expectedValue?: string }
Output: { pass: boolean, actualValue: string, message: string }

Example:
  Input: { path: "$.message", expectedValue: "reset link has been sent" }
  Output: { pass: true, actualValue: "reset link has been sent", message: "JSON assertion passed" }
```

### Shell Commands (Limited)

```
Tool: cli/run_command
Input: { command: string, timeout?: number }
Output: { status: number, stdout: string, stderr: string, durationMs: number }

Restrictions:
  - Only whitelisted commands (e.g., curl, jq, grep, date, sleep)
  - Cannot contain: rm, dd, : (redirect), | (pipe), & (background), ; (chain)
  - All arguments are escaped/validated

Example:
  Input: { command: "curl -X GET https://api.staging.example.com/health" }
  Output: { status: 0, stdout: '{"healthy": true}', stderr: "", durationMs: 320 }
```

### Context Management

```
Tool: cli/set_header
Input: { name: string, value: string }
Output: { status: "success" }

Tool: cli/set_bearer_token
Input: { token: string }
Output: { status: "success" }

Tool: cli/clear_headers
Input: {}
Output: { status: "success" }

Example:
  Input: { name: "Authorization", value: "Bearer eyJhbGc..." }
  Output: { status: "success" }
```

---

## Example MCP Tool Definition (JSON Schema)

```json
{
  "name": "cli/http_request",
  "description": "Make an HTTP request to a whitelisted endpoint",
  "inputSchema": {
    "type": "object",
    "properties": {
      "method": {
        "type": "string",
        "enum": ["GET", "POST", "PUT", "DELETE", "PATCH"],
        "description": "HTTP method"
      },
      "url": {
        "type": "string",
        "description": "URL to request. Must match a whitelisted endpoint pattern."
      },
      "headers": {
        "type": "object",
        "description": "HTTP headers"
      },
      "body": {
        "oneOf": [
          { "type": "object" },
          { "type": "string" }
        ],
        "description": "Request body"
      },
      "timeout": {
        "type": "number",
        "description": "Timeout in milliseconds. Default: 10000, Max: 30000"
      }
    },
    "required": ["method", "url"]
  }
}
```

---

## Endpoint Allowlisting

Before running, define which endpoints the agent can access:

```yaml
# allowlist.yaml
endpoints:
  - pattern: https://api.staging.example.com/password-reset
    methods: [GET, POST]
    description: Password reset request
  
  - pattern: https://api.staging.example.com/reset
    methods: [POST]
    description: Confirm password reset with token
  
  - pattern: https://api.staging.example.com/login
    methods: [POST]
    description: Login (for test setup)
  
  - pattern: https://api.staging.example.com/health
    methods: [GET]
    description: Health check

  # DO NOT allow:
  # - https://api.production.example.com (no prod access)
  # - https://api.staging.example.com/admin (admin endpoints)
  # - https://api.staging.example.com/user/{userId} (PII, unless explicitly needed)
```

Any request to an unlisted endpoint is rejected before sending.

---

## Usage in a Test

When an agent executes a test, it might:

```
Agent receives: "Test: user can request password reset"
Agent reads test oracle: "POST /password-reset with email, expect 200 + confirmation message"

Agent calls MCP:
  1. cli/http_request {
       method: "POST",
       url: "https://api.staging.example.com/password-reset",
       body: { email: "test@example.com" }
     }
  2. cli/assert_status { expectedStatus: 200 }
  3. cli/assert_json_path { path: "$.message", expectedValue: "reset link has been sent" }

MCP returns results for each step.
Agent collects into execution report:
  "✓ PASS: User can request password reset (HTTP 200, correct message)"
```

---

## Implementation Notes

### Timeout Management
- Default: 10 seconds
- Max: 30 seconds
- Agent should be explicit about slow endpoints

### Response Handling
- JSON responses are parsed automatically
- XML/HTML responses returned as string
- Binary responses rejected (with error)

### Error Handling
```
if status >= 500:
  { pass: false, message: "Server error (HTTP {status})" }

if status >= 400:
  { pass: false, message: "Client error (HTTP {status}). Response: {body}" }

if timeout:
  { pass: false, message: "Request timed out after {timeout}ms" }
```

Agent decides whether to retry, escalate, or fail the test.

### Logging
Every request is logged with full details:
```
[2025-06-10 14:23:45] cli/http_request POST https://api.staging.example.com/password-reset
  Body: { email: "test@example.com" }
  → 200 OK (145ms)
  Response: { message: "reset link has been sent", resetUrl: "..." }
```

This log is the audit trail and can be included in the execution report.

---

## Security Hardening

See `/security/mcp-security-guide.md` for:
- How to define safe endpoint allowlists
- Handling API keys and bearer tokens
- Detecting exfiltration attempts
- Audit trail best practices

---

## Token Cost

CLI MCP is **lower token cost** than Playwright because:
- Structured JSON responses (compact)
- No DOM parsing or browser overhead
- Quick request/response cycle

**Estimate:**
- Simple GET + assertion: 100-200 tokens
- POST + response parsing + assertions: 200-400 tokens
- Multiple chained requests: 300-600 tokens

**Compared to Playwright:**
- CLI: 200 tokens per test assertion
- Playwright: 500+ tokens per test assertion (2-3x more expensive)

---

## When to Use CLI MCP

✓ **Use when:**
- Testing REST/HTTP APIs
- Need to verify response structure and values
- Need low token cost (high volume of tests)
- Testing CLI tools or custom commands
- API-first testing (no UI verification)

✗ **Don't use when:**
- Need to verify UI rendering (use Playwright instead)
- Testing non-HTTP protocols (gRPC, GraphQL, WebSocket — use specialized MCPs)
- Need screenshots or visual debugging
- Testing client-side logic (JavaScript, browser state)

---

## Links

- **MCP Overview:** [README.md](README.md)
- **Playwright MCP Pattern:** [playwright-mcp-pattern.md](playwright-mcp-pattern.md)
- **Security:** [/security/mcp-security-guide.md](../security/mcp-security-guide.md)
- **Token Tradeoffs:** [/guides/token-tradeoffs.md](../guides/token-tradeoffs.md)

---

*Phase 2: MCP patterns. Phase 3+ will add specialized servers (gRPC, GraphQL, mobile, etc.)*


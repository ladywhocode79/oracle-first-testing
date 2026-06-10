# Playwright MCP Pattern

A Model Context Protocol server for controlled browser automation in testing.

---

## What It Does

Exposes Playwright operations as MCP tools, but with scoping:

✓ **Allowed:** click, type, select, screenshot, read text, get attributes, wait for elements, navigate
✗ **Blocked:** executeScript, setViewportSize (can be used to hide elements), accessStorage, modifyCookie (directly)

This prevents the agent from:
- Injecting JavaScript (can't call window.fetch directly)
- Reading/modifying localStorage or sessionStorage
- Changing cookies directly (though agent can authenticate via login flow)
- Resizing viewport to hide elements from view

But allows the agent to:
- Interact with the UI as a normal user would
- Verify what's displayed on the page
- Screenshot for visual debugging

---

## MCP Tool Definitions

The server exposes these tools:

### Navigation
```
Tool: playwright/navigate
Input: { url: string }
Output: { status: "navigated" | "timeout" | "failed", currentUrl: string }

Example:
  Input: { url: "https://staging.example.com/password-reset" }
  Output: { status: "navigated", currentUrl: "https://staging.example.com/password-reset" }
```

### Element Interaction
```
Tool: playwright/click
Input: { selector: string, timeout?: number }
Output: { status: "success" | "timeout" | "not_found", message: string }

Tool: playwright/type
Input: { selector: string, text: string, timeout?: number }
Output: { status: "success" | "timeout" | "not_found", message: string }

Tool: playwright/select
Input: { selector: string, value: string, timeout?: number }
Output: { status: "success" | "timeout" | "not_found", message: string }

Example:
  Input: { selector: "input[name='email']", text: "test@example.com" }
  Output: { status: "success", message: "Typed into email input" }
```

### Assertions
```
Tool: playwright/assert_text
Input: { selector: string, expectedText: string, timeout?: number }
Output: { status: "pass" | "fail" | "timeout" | "not_found", actualText: string, message: string }

Tool: playwright/assert_visible
Input: { selector: string, timeout?: number }
Output: { status: "pass" | "fail" | "timeout", message: string }

Tool: playwright/assert_enabled
Input: { selector: string, timeout?: number }
Output: { status: "pass" | "fail" | "timeout", message: string }

Example:
  Input: { selector: ".success-message", expectedText: "reset link has been sent" }
  Output: { status: "pass", actualText: "reset link has been sent", message: "Text assertion passed" }
```

### Page State
```
Tool: playwright/get_text
Input: { selector: string, timeout?: number }
Output: { status: "success" | "not_found", text: string }

Tool: playwright/get_attribute
Input: { selector: string, attribute: string, timeout?: number }
Output: { status: "success" | "not_found", value: string }

Tool: playwright/screenshot
Input: { path?: string, fullPage?: boolean }
Output: { status: "success" | "failed", imagePath: string }

Tool: playwright/wait_for_url
Input: { urlPattern: string | regex, timeout?: number }
Output: { status: "success" | "timeout", currentUrl: string }

Example:
  Input: { selector: ".error-message", timeout: 2000 }
  Output: { status: "success", text: "This field is required" }
```

### Context Management
```
Tool: playwright/set_auth
Input: { username: string, password: string }
Output: { status: "success" | "failed", message: string }

Tool: playwright/clear_cookies
Input: {}
Output: { status: "success" }

Example:
  Input: { username: "test@example.com", password: "TestPass123" }
  Output: { status: "success", message: "Logged in successfully" }
```

---

## Example MCP Tool Definition (JSON Schema)

```json
{
  "name": "playwright/click",
  "description": "Click an element on the page by CSS selector",
  "inputSchema": {
    "type": "object",
    "properties": {
      "selector": {
        "type": "string",
        "description": "CSS selector for the element to click"
      },
      "timeout": {
        "type": "number",
        "description": "Maximum time to wait for element, in milliseconds. Default: 5000"
      }
    },
    "required": ["selector"]
  }
}
```

---

## Usage in a Test

When an agent is executing a test against the oracle, it might:

```
Agent receives: "Test: user can request password reset"
Agent reads test oracle: "User enters email, clicks reset button, sees confirmation"

Agent calls MCP:
  1. playwright/navigate { url: "https://staging.example.com/reset" }
  2. playwright/type { selector: "input[name='email']", text: "test@example.com" }
  3. playwright/click { selector: "button[type='submit']" }
  4. playwright/wait_for_url { urlPattern: "/reset-requested" }
  5. playwright/assert_text { selector: ".success-message", expectedText: "reset link has been sent" }

MCP returns results for each step.
Agent collects into execution report:
  "✓ PASS: User can request password reset (confirmation message displayed)"
```

---

## Implementation Notes

### Timeouts
- Default: 5 seconds for element visibility
- Long operations (email arrival): up to 30 seconds
- Agent should be explicit about timeout needs

### Error Handling
- If selector doesn't match: return `{ status: "not_found", message: "... no elements matched ..." }`
- If action times out: return `{ status: "timeout", message: "... waited X seconds ..." }`
- Agent decides what to do (retry, fail test, escalate)

### Logging
Every MCP call is logged:
```
[2025-06-10 14:23:45] playwright/navigate { url: "https://staging.example.com/reset" }
  → { status: "navigated", currentUrl: "https://staging.example.com/reset" }

[2025-06-10 14:23:46] playwright/type { selector: "input[name='email']", text: "test@example.com" }
  → { status: "success", message: "Typed into email input" }
```

This log is the audit trail. Auditors can verify: "Here's exactly what the agent did."

### Session Management
- One browser session per test run (or per test, depending on needs)
- `playwright/set_auth` handles login once; subsequent tests reuse the session
- `playwright/clear_cookies` for cleanup between test sections

---

## Security Hardening

See `/security/mcp-security-guide.md` for:
- What data is exposed in Playwright output (DOM content, URLs, text)
- How to handle credentials safely
- When to use service accounts vs. user accounts
- Audit trail best practices

---

## Token Cost

Playwright MCP is **higher token cost** than CLI because:
- DOM dumps can be large (full page HTML)
- Screenshot descriptions are verbose
- Waiting/polling generates multiple round-trips

**Estimate:**
- Simple assertion (click + check text): 200-500 tokens
- Complex interaction (type + wait + verify): 500-1000 tokens
- Screenshot + analysis: 2000-5000 tokens

**Optimization:**
- Use specific selectors (CSS, not XPath or text-based)
- Don't screenshot unless needed
- Batch assertions (one click, multiple checks on result)

---

## When to Use Playwright MCP

✓ **Use when:**
- Testing a web UI
- Need to verify user-facing behavior (layout, text, styling)
- Screenshots help debug
- Need to test client-side interactions (autocomplete, validation, etc.)

✗ **Don't use when:**
- Testing a REST API (use CLI MCP instead, lower token cost)
- No UI verification needed
- Need high-speed execution (Playwright startup = overhead)
- Testing a non-web system (gRPC, databases, etc.)

---

## Links

- **MCP Overview:** [README.md](README.md)
- **CLI MCP Pattern:** [cli-mcp-pattern.md](cli-mcp-pattern.md)
- **Security:** [/security/mcp-security-guide.md](../security/mcp-security-guide.md)
- **Token Tradeoffs:** [/guides/token-tradeoffs.md](../guides/token-tradeoffs.md)

---

*Phase 2: MCP patterns. Phase 3+ will add specialized servers (gRPC, GraphQL, mobile, etc.)*


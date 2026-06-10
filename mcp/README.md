# MCP for Test Execution

Model Context Protocol (MCP) servers provide sandboxed, scoped access to testing tools.

Instead of the test agent directly calling APIs or controlling browsers, it goes *through* an MCP server that:
- Enforces what operations are allowed (read-only vs. mutating)
- Tracks every action the agent takes (audit trail)
- Translates high-level test commands into tool actions
- Handles credential/token management safely

## Why MCP?

**Without MCP:** Agent has full access to a browser or API. It could:
- Extract secrets from the page
- Delete data
- Modify production data (if pointing to prod)
- Exfiltrate information

**With MCP:** Agent has *scoped* access. It can:
- Click, type, assert text (not inject JavaScript, not access storage)
- Read responses from APIs (not modify sensitive fields without approval)
- Screenshot (not access browser memory)
- Every action is logged and reversible

## Two MCP Tracks

### Track 1: Playwright MCP
For **UI/browser testing**. The agent controls a browser via Playwright, but through an MCP server that:
- Filters what the agent can do (click, type, read text, screenshot)
- Logs all interactions
- Manages authentication (agent doesn't see the password)

**Use when:**
- Testing a web UI
- Need to verify user-facing behavior
- Need screenshots for debugging

**Token cost:** Higher (Playwright generates verbose logs)

### Track 2: CLI MCP
For **API/command-line testing**. The agent makes HTTP requests or shell commands, but through an MCP server that:
- Validates requests before sending
- Only allows specific endpoints/operations
- Manages credentials safely
- Logs all requests/responses

**Use when:**
- Testing APIs
- Testing CLI tools
- Need fast, low-token-cost execution
- No UI verification needed

**Token cost:** Lower (structured JSON responses)

## How They Connect to `/prompts/execute/`

The execute stage prompts use MCP to actually run tests:

```
Prompt: "Run this test suite against the oracle"
  ↓
Agent reads test code + oracle
  ↓
Agent calls Playwright MCP: "Click the reset button"
Agent calls CLI MCP: "POST /reset with token X"
  ↓
MCP executes, returns result
Agent collects results and builds execution report
```

## Structure of This Directory

- **playwright-mcp-pattern.md** — How to structure a Playwright MCP server
- **cli-mcp-pattern.md** — How to structure a CLI MCP server
- **/examples/** — Working example MCP definitions
  - playwright-server-example.md
  - cli-server-example.md
- **security-guide.md** — (extends /security/) MCP-specific threat model and hardening

## Next Steps

1. Read the pattern docs to understand structure
2. Look at examples to see concrete definitions
3. Connect to `/prompts/execute/` to see how agents use MCP
4. Reference `/guides/token-tradeoffs.md` to decide which track to use

---

*Phase 2: MCP foundation. Phase 3+ will add more specialized MCP servers (GraphQL, gRPC, etc.)*


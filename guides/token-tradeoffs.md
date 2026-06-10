# Token Tradeoffs: Which Execution Track to Use

When running tests via an LLM agent, you have choices. Each has different token costs, latency, and capabilities.

---

## The Three Options

### Option A: CLI MCP (Cheapest)
Agent makes HTTP requests via CLI MCP, parses JSON responses.

**Token cost per test assertion:** 100-200 tokens  
**Latency:** 200-500ms per request  
**Best for:** APIs, quick checks, high volume

**Example:**
```
Agent: POST /password-reset, expect HTTP 200
MCP: { status: 200, body: { message: "..." } }
Agent assertion: ✓ PASS (200 tokens total)
```

---

### Option B: Playwright MCP (Medium)
Agent drives a browser via Playwright MCP, clicks, types, reads the DOM.

**Token cost per test assertion:** 500-1000 tokens  
**Latency:** 1-3 seconds per interaction  
**Best for:** UI testing, user-facing behavior, visual verification

**Example:**
```
Agent: Click reset button, verify success message
MCP: [navigate, click, wait, read text]
Agent assertion: ✓ PASS (700 tokens total)
```

---

### Option C: Direct Test Execution (No MCP)
Agent doesn't use MCP. Instead, you provide test code directly, and the agent just reports results.

**Token cost per test:** 50-100 tokens  
**Latency:** Depends on test framework (seconds to minutes)  
**Best for:** Already-written test suites, regression runs, CI/CD integration

**Example:**
```
You: Run pytest tests/test_reset.py
Agent: pytest [results]
Agent report: 25 passed, 2 failed (75 tokens total)
```

---

## Decision Matrix

| Scenario | Option | Why | Token Cost | Notes |
|----------|--------|-----|-----------|-------|
| **Testing a REST API** | CLI MCP | Structured responses, fast, cheap | Low | Go here first |
| **Testing a web UI** | Playwright MCP | Need browser interaction | Medium | Only way to test UI |
| **Testing CLI tool** | CLI MCP or Direct | If simple, CLI MCP. If complex, direct. | Low-Medium | Depends on tool |
| **End-to-end user flow** | Playwright MCP + CLI MCP | UI + API verification | Medium | Split: UI via Playwright, APIs via CLI |
| **Existing test suite (pytest, Jest)** | Direct test execution | Already coded, just run it | Very Low | Fastest option |
| **Quick smoke test** | CLI MCP | One API call, one assertion | Very Low | Sub-second turnaround |
| **Full regression** | Direct test execution | High volume, already coded | Very Low | Scales well |
| **Complex business logic** | Direct test execution | Too many branches for agent to explore | Very Low | Let the code decide |

---

## Cost Estimation

### Scenario: Testing Password Reset Feature

**Option A: CLI MCP**
- Navigate to login page: 1 request (150 tokens)
- POST /login: 1 request (200 tokens)
- POST /password-reset: 1 request (200 tokens)
- Assert response: (50 tokens)
- Check email via mock API: 1 request (150 tokens)
- **Total: ~750 tokens**

**Option B: Playwright MCP**
- Navigate to reset page: 1 interaction (300 tokens)
- Type email: 1 interaction (300 tokens)
- Click submit: 1 interaction (300 tokens)
- Wait for success message: 1 wait (300 tokens)
- Assert message visible: 1 assertion (200 tokens)
- Screenshot for proof: 1 screenshot (2000 tokens)
- **Total: ~3400 tokens (4.5x more expensive)**

**Option C: Direct Test Execution**
- Run pytest (test already written):
```python
def test_user_can_request_reset():
  response = post("/password-reset", {"email": "test@example.com"})
  assert response.status_code == 200
  assert "reset link has been sent" in response.json()["message"]
```
- **Total: ~100 tokens to report the result**

---

## Token Cost Breakdown

### CLI MCP
```
Per request:
  - Formatting request: 50 tokens
  - HTTP call (overhead): 50 tokens
  - Parsing response: 50 tokens
  - Assertion logic: 50 tokens
  ─────────────
  Total: ~200 tokens per API call
```

### Playwright MCP
```
Per interaction:
  - Formatting command: 100 tokens
  - Playwright execution (logs): 200 tokens
  - DOM parsing/analysis: 200 tokens
  - Assertion logic: 100 tokens
  ─────────────
  Total: ~600 tokens per interaction
  (3x CLI cost)
```

### Direct Test Execution
```
Once:
  - Run test suite: 50 tokens
  - Parse test results: 30 tokens
  - Format report: 20 tokens
  ─────────────
  Total: ~100 tokens per test suite run
  (regardless of # tests)
```

---

## When NOT to Use Each Option

### Don't use CLI MCP when:
- You need to test UI behavior (clicks, typing, layout)
- API responses are huge (JSON files with millions of elements)
- You need authentication that requires browser (SSO, SAML)
- Response time matters more than cost (Playwright is faster for complex flows)

### Don't use Playwright MCP when:
- Testing APIs with no UI
- You have thousands of tests (too expensive at scale)
- Tests need to run offline (Playwright requires a browser instance)
- Edge case: very low bandwidth (browser logs are verbose)

### Don't use Direct Test Execution when:
- You need the agent to explore and diagnose
- Test code doesn't exist yet (you're writing tests)
- You need rich reporting (screenshots, step-by-step logs)
- You need the agent to decide what to test

---

## Hybrid Approach (Recommended)

**Best of all worlds:**

```
Phase 1: Use Playwright MCP
  ├─ Agent navigates to /reset page
  ├─ Agent types email
  ├─ Agent clicks submit
  └─ Agent screenshots success (for proof)

Phase 2: Use CLI MCP
  ├─ Agent checks email delivery (via mock API)
  ├─ Agent extracts reset link
  └─ Agent verifies token expires after 24h (via API calls)

Phase 3: Use Direct Test Execution
  ├─ Run full pytest suite
  └─ Report pass/fail metrics
```

**Token cost:** ~1500 tokens (Playwright screenshot) + ~500 tokens (CLI assertions) + ~100 tokens (test report) = ~2100 tokens

**Value:**
- ✓ Verifies UI works
- ✓ Verifies email backend works
- ✓ Verifies full suite passes
- ✓ Cheaper than Playwright-only
- ✓ More thorough than API-only

---

## Pricing Impact (at scale)

Assuming Claude 3 pricing (~$0.003 per 1K tokens):

| Approach | Tokens per test | Cost per 100 tests | Cost per 1000 tests |
|----------|-----------------|------------------|-------------------|
| CLI MCP | 200 | $0.06 | $0.60 |
| Playwright MCP | 600 | $0.18 | $1.80 |
| Direct execution | 100 | $0.03 | $0.30 |
| Hybrid | 300 | $0.09 | $0.90 |

**At scale (1000s of tests), direct execution is 18-20x cheaper.**

But if you *don't have tests written yet*, the agent + MCP approach is the only way to go.

---

## Recommendations by Use Case

### New Feature (Tests Don't Exist)
→ **Playwright MCP + CLI MCP**
- Agent explores the feature
- Agent writes/executes tests on-the-fly
- Expensive upfront, but worth it for first-time coverage
- Estimated cost: $5-10 per feature

### Regression Testing (Tests Exist)
→ **Direct Test Execution**
- Tests are already coded (pytest, Jest, etc.)
- Agent just runs them and reports
- Cheapest option
- Estimated cost: $0.30 per regression run

### UI Feature Testing
→ **Playwright MCP**
- User-facing changes
- Screenshots help explain what changed
- Worth the extra token cost for credibility
- Estimated cost: $2-5 per feature

### API Integration Testing
→ **CLI MCP**
- Fast, cheap, structured
- Only way to test APIs without UI
- Estimated cost: $0.50-1.50 per test

### CI/CD Integration
→ **Direct Test Execution**
- Run existing tests on every commit
- Cheapest, fastest
- Agent only reports results
- Estimated cost: $0.10 per CI run

---

## Making the Decision

**Ask yourself:**

1. **Do tests already exist?**  
   YES → Use direct execution  
   NO → Use MCP (Playwright + CLI)

2. **Does this test a UI?**  
   YES → Use Playwright MCP  
   NO → Use CLI MCP or direct execution

3. **How many tests?**  
   <50 → MCP is fine  
   >100 → Use direct execution if possible

4. **How often do you run?**  
   Once (new feature) → MCP is acceptable  
   Daily (regression) → Direct execution is mandatory

5. **Do you need screenshots/visual proof?**  
   YES → Playwright MCP  
   NO → CLI MCP or direct execution

---

## Links

- **CLI MCP Pattern:** [/mcp/cli-mcp-pattern.md](../mcp/cli-mcp-pattern.md)
- **Playwright MCP Pattern:** [/mcp/playwright-mcp-pattern.md](../mcp/playwright-mcp-pattern.md)
- **Execute Prompts:** [/prompts/execute/](../prompts/execute/)

---

*Phase 2: Token tradeoffs. Use this guide to make informed decisions about your execution strategy.*


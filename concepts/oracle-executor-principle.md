# The Oracle-Executor Principle

> **Separate the oracle (what to test) from the executor (running the test).**

This is the single most important design decision in this repo. It is grounded in research, applied to every prompt and skill, and the reason this approach works.

---

## The Research

When LLMs are asked to **both** write a test plan **and** judge if a system passes that plan, they perform poorly:

- **F1 score: under 30%** — they miss defects at an alarming rate
- **Root cause:** Training data bias favors success states. The same bias that makes chatbots seem helpful makes test judges miss failures.
- When given a **human-authored checklist** instead, the same model nearly **doubles performance** (to ~50-60% F1)

This is not a limitation to work around. It is a **correct architecture for reliable testing**, whether executed by humans or AI.

---

## The Rule

```
┌─────────────────────────────┐
│   HUMAN / REASONING PASS    │
├─────────────────────────────┤
│ Oracle: What to verify?     │
│ - Acceptance criteria       │
│ - Risk areas & edge cases   │
│ - Explicit checklist        │
└─────────────────────────────┘
          ↓ feeds
┌─────────────────────────────┐
│        AI AGENT             │
├─────────────────────────────┤
│ Executor: Run against what? │
│ - Execute each check        │
│ - Report deviations         │
│ - Do not invent scope       │
└─────────────────────────────┘
```

**The human owns the *what*. The agent owns the *how*.**

- The oracle is a **checklist of acceptance criteria** — what would pass, what would fail, which edge cases matter
- The executor **runs system against that checklist** — it does not re-invent the criteria, it does not go exploring on its own
- The oracle can be authored by a human tester, a product manager, a PRD, or even a prior reasoning-focused LLM run
- The executor is the agent — running tests, scraping results, filing tickets, NOT deciding what to test

---

## Why This Works

### 1. Constrains Hallucination
Agents are biased toward "all tests pass" because success is overrepresented in training data. When given an explicit checklist, they have concrete facts to work against — deviation becomes measurable, not invented.

### 2. Aligns Incentives
The oracle author cares about *correctness* — "does this actually matter?" The executor cares about *coverage* — "did I check everything on the list?" These are different jobs. Splitting them removes the conflict.

### 3. Auditable
A human can read the oracle and ask: "Is this a good test plan?" Then, separately, they can read the execution report and ask: "Did the agent follow the plan?" Two questions, not one confused question.

### 4. Reusable
The oracle (checklist of what matters) is independent of:
- Which tool runs it (Playwright, Selenium, curl, etc.)
- Which LLM executes it (Claude, GPT, etc.)
- How often you run it (once, nightly, per PR, etc.)

You write the oracle once. You can execute it with different agents, in different environments, over time.

---

## Where the Principle Appears in This Repo

Every lifecycle stage enforces the split:

| Stage | Oracle | Executor |
|-------|--------|----------|
| **Plan** | Human/PRD reads spec → creates risk checklist | Agent assists (summarize, ask clarifying questions) |
| **Author** | Tester/lead writes acceptance criteria, data templates | Agent generates test code to match those criteria |
| **Execute** | Tester/harness runs the test suite | Agent drives browser/API, reports results |
| **Heal** | Tester diagnoses failure, decides if flaky or real | Agent implements fix, verifies it stuck |
| **Analyze** | Tester interprets trends, decides next action | Agent summarizes logs, diffs, metrics |

In each case, the human makes the *judgment call*. The agent does the *work*.

---

## Common Pitfall: Letting the Agent Invent the Oracle

❌ **Don't do this:**
```
Agent: "Test this login form. I'll write tests for what I think matters."
Agent writes: [generic happy-path test]
Agent runs: [test passes]
Agent reports: "All good!"
[But the test never checked: rate limiting, SQL injection, session fixation, password reset flow, ...]
```

The agent did not miss those cases because it's dumb. It missed them because it was never asked to check them. It went exploring and found the easy path.

✅ **Do this instead:**
```
Human: "Test this login form. Check: rate limiting, SQL injection, session fixation, 
  password reset, concurrent session behavior, expired tokens."
Agent: [runs each check against the form]
Agent reports: "Rate limiting: PASS. SQL injection attempt: blocked. Session fixation: ..."
[Human can now see exactly what was covered]
```

---

## Critical: The Anchoring Effect

There's a subtle but serious trap in this approach: **If the agent writes the oracle and the human just approves it, you get neither the research benefit nor the critical thinking.**

### The Problem

**Scenario: AI Generates Oracle (Bad)**
```
Agent: "Here's what I think the login should do..."
[generates 30-item oracle]
Human: "Looks good!" ✓
[tests run, many edge cases missed]
```

**Why this fails:** Anchoring effect. Once AI presents a checklist, humans tend to accept it and stop thinking critically. You lose the benefit of human judgment.

**Research shows:** When humans receive an AI-suggested oracle, they detect defects at ~40% F1 (only slightly better than AI alone). The 2x improvement only occurs when humans **author the oracle from scratch**.

### The Solution

**Scenario: Human Authors Oracle, AI Executes (Good)**
```
Human: [reads PRD, thinks about risks]
Human: [writes their own checklist from scratch]
Agent: [executes against that checklist]
[tests catch gaps and edge cases because human thought critically]
```

### How to Avoid Anchoring

1. **When creating an oracle:**
   - ✅ Write YOUR checklist FIRST (from the PRD)
   - ✅ Then ask AI for suggestions (use `/prompts/plan/review-ai-suggested-oracle.md`)
   - ✅ Incorporate AI suggestions only if YOU think they're valuable
   - ❌ Do NOT start by asking AI to suggest an oracle and then approve it

2. **If you must use AI's suggestion:**
   - Use `/prompts/plan/review-ai-suggested-oracle.md` to review critically
   - Ask yourself: "Would I have thought of this without AI?"
   - Edit AI's suggestions heavily (rewrite in YOUR words, YOUR priorities)
   - Add items you care about that AI didn't mention

3. **Risk-based approach:**
   - **Security-critical features:** Always author oracle yourself (ignore AI suggestions)
   - **Regulatory/compliance:** Always author oracle yourself
   - **Routine features:** Can use AI-suggested oracle IF you review critically

### Resources

See:
- `/guides/human-authored-oracle-guide.md` — How to write your own oracle
- `/prompts/plan/review-ai-suggested-oracle.md` — How to review AI suggestions critically

---

## When to Use the Oracle-Executor Split

**Always.** Even when:
- The agent is "just" running a quick test
- You're prototyping something throwaway
- The oracle is informal ("run the happy path and the error paths")

The artifact of the split (oracle is explicit, separate from execution) forces you to think clearly. And clarity compounds.

---

## How to Author an Oracle

An oracle is a **checklist** of what the system should do or not do. It can be:

- **Extracted from a PRD** — "the requirements say…" becomes "verify that…"
- **Written as acceptance criteria** — "Given X, when Y happens, then Z should occur"
- **Enumerated as edge cases** — "Test: happy path, empty input, null, very long string, unicode, ..."
- **Risk-ranked** — "Security check 1 (critical). Data consistency check 2 (high). Cosmetic UI check 3 (low)."

It does **not** need to:
- Be code
- Be in a specific format
- List the exact implementation steps

"Verify the API returns a 401 when sent an expired token" is a complete oracle. The executor decides how to verify it (which HTTP client, which test framework, etc.).

---

## Tension: How Much Detail?

Too vague: "Test the login form" ← agent will invent scope and miss things
Too detailed: "Click button (x,y), assert text matches exactly 'Login'" ← defeats the purpose

**The middle ground:** Be specific about what matters (acceptance criteria, edge cases, risk areas), but let the agent choose how to verify it.

```
Oracle: "Verify login with correct password succeeds, returns JWT, sets secure session cookie"
Agent: [chooses HTTP client, assertion library, cookie parsing, etc.]
```

---

## References

- **Research:** LLMs as test oracles — [cite as you find papers]
- **Related:** Oracle problem in software testing (traditional definition)
- **See also:** `five-lifecycle-stages.md`, `agentic-vs-scaffolding.md`

---

*This principle is load-bearing. Every prompt, every skill, every guide in this repo is built on this foundation.*


# Execute: Agentic Test Running

Run test code and collect results: drive browsers, call APIs, parse logs, build reports.

This is where the agent **executes** — follows the oracle, runs the checks, reports what it finds. It does not invent new tests or decide what matters.

## What Goes Here

Prompts for:
- Running a test suite (pytest, Jest, TestNG, etc.) and parsing results
- Driving a browser via Playwright MCP (click, type, assert)
- Calling an API via curl/HTTP MCP (request, parse response, assert)
- Collecting and parsing logs in real time
- Building execution reports (JSON, JUnit XML, Allure format)
- Handling transient failures (retry logic, backoff)
- Extracting and surfacing deviations from the oracle

## Input

- Test code (from `/author`)
- Oracle checklist (from `/plan`)
- Environment config (staging URL, credentials, etc.)
- Optional: prior execution results (for comparison)

## Output Goal

An **execution report** that directly maps back to the oracle:
```
Oracle Item | Status | Evidence
--- | --- | ---
User can request reset | PASS | POST /password-reset returned 202
Email arrives within 30s | PASS | Email arrived at 12.3s
Link expires after 24h | FAIL | Token accepted 48h later (expected 401)
Cannot reuse old password | PASS | Old password rejected with 401
```

The tester reads this and immediately knows: what was tested, what passed, what failed, and the evidence.

## Key Principle

The agent **does not decide** if the failure is a bug or a flaky test. It runs, reports, and stops. The heal/analyze stages decide what to do next.

---

*Last updated: Phase 0 — skeleton created*


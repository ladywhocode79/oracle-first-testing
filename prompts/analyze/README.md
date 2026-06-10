# Analyze: Results to Insight

Turn test results into actionable insight: failure trends, coverage gaps, risk exposure, what to test next.

## What Goes Here

Prompts for:
- Summarizing test results (pass rate, fail rate, flaky rate)
- Triaging failures by area, priority, or root cause
- Detecting new failures vs. known failures vs. flaky noise
- Performance regression detection (latency, throughput)
- Coverage analysis (what areas are under-tested?)
- Risk exposure (critical tests failing? Security tests passing?)
- Recommending what to test next

## Input

- Execution report (from `/execute`)
- Historical results (prior runs, trends)
- Optional: code changes (what changed since last run?)
- Optional: risk ranking (from `/plan`)

## Output Goal

An **insight report** that answers:

```
## Summary
- 87/100 tests pass (87%)
- 5 new failures (in checkout flow)
- 3 known flaky (email delivery, external API)
- Performance: 2% slower than baseline

## Triage
Critical (fix immediately):
  - test_checkout_payment_fails_ungracefully (3 failures)
  - test_inventory_decrement_race_condition (intermittent)

High (address this cycle):
  - test_promo_code_invalid_format (1 failure)
  - test_refund_state_machine (state inconsistent)

Low (monitor):
  - test_email_delivery (flaky, but eventually passes)
  - test_analytics_tracking (performance regression)

## Gaps
- No tests for concurrent checkout with same coupon
- No tests for payment failure + inventory recovery
- Recommendation: add 2-3 tests to cover these

## Risk Exposure
- 5 critical tests pass (payment, auth, inventory) ✓
- 0 security tests present for password reset ✗
- Recommendation: add OWASP Top 10 coverage before launch
```

The tester uses this to decide: is this good enough to ship? What fails next?

## Key Principle

Analysis is **about the system**, not about the agent. The report should be actionable by any human, regardless of which tool ran the tests.

---

*Last updated: Phase 0 — skeleton created*


# Oracle-First Workflow — Implementation Guide

How to orchestrate the full pipeline: PRD → Oracle → Tests → Execution → Report.

---

## The Pattern

```
Human provides: PRD
        ↓
Claude Phase 1: Extract oracle (what to test)
        ↓
Claude Phase 2: Generate tests (how to test)
        ↓
Claude Phase 3: Execute tests (run them)
        ↓
[If failures]
Claude Phase 4: Diagnose & fix (why did it fail?)
        ↓
Claude Phase 5: Analyze readiness (ship?)
        ↓
Human decides: Ship or iterate
```

---

## Running the Workflow

### Option A: Interactive (Best for Learning)

Suitable for: First-time features, learning the process, high control

**Time:** 90 minutes per feature  
**Tokens:** 3-4k per feature  
**Cost:** ~$0.01 per feature

**Steps:**

1. **Read the PRD**
   ```
   Get: requirements document, acceptance criteria, edge cases
   ```

2. **Phase 1: Extract Oracle**
   ```
   Prompt: /prompts/plan/extract-oracle-from-prd.md
   Input: PRD + context (domain, risk level)
   Output: Checklist of testable claims
   
   You review: "Does this oracle match what we care about?"
   You approve: "Yes, use this."
   
   Time: 10 min
   Tokens: 200
   ```

3. **Phase 2: Author Tests**
   ```
   Prompt: /prompts/author/generate-tests-from-oracle.md
   Input: Oracle + language + framework
   Output: Test code
   
   You review: "Do these tests match the oracle?"
   You approve or request changes: "Add test X, remove Y"
   
   Time: 15 min
   Tokens: 300
   ```

4. **Phase 3: Execute Tests**
   ```
   Setup: Create MCP allowlist (whitelisted endpoints)
   
   Prompt: /prompts/execute/run-tests-and-report.md
   Input: Test code + environment + MCP setup
   Output: Execution report (pass/fail + evidence)
   
   You review: "What failed? Is it important?"
   
   Time: 5 min
   Tokens: 1200
   ```

5. **Phase 4: Heal (if failures)**
   ```
   Prompt: /prompts/heal/diagnose-test-failure.md
   Input: Failed test + logs + oracle
   Output: Diagnosis (bug in test? bug in code? bug in oracle?)
   
   You decide: Fix product or fix test?
   Product team fixes it
   Re-run Phase 3
   
   Time: 15 min
   Tokens: 400
   ```

6. **Phase 5: Analyze**
   ```
   Prompt: /prompts/analyze/summarize-results.md
   Input: Execution report + diagnoses
   Output: Readiness assessment (ready to ship?)
   
   You review: "Is this safe to ship?"
   You decide: Ship or iterate
   
   Time: 10 min
   Tokens: 200
   ```

---

### Option B: Scripted (Best for Automation)

Suitable for: Many features, CI/CD integration, high throughput

**Time:** 30 minutes per feature (with human review gate)  
**Tokens:** 3-4k per feature  
**Cost:** ~$0.01 per feature

**Pseudo-code:**

```python
def oracle_first_workflow(prd_text, feature_name, environment):
    """Run the full oracle-first workflow"""
    
    # Phase 1: Plan
    print("Phase 1: Extracting oracle from PRD...")
    oracle = claude.prompt(
        template="/prompts/plan/extract-oracle-from-prd.md",
        input={
            "prd": prd_text,
            "domain": "e-commerce",
            "risk_level": "critical"
        }
    )
    print(f"Oracle:\n{oracle}")
    
    # Human: Review and approve oracle
    if not human_approves(oracle):
        print("Oracle rejected. Iterate.")
        return
    
    # Phase 2: Author
    print("\nPhase 2: Generating test code...")
    test_code = claude.prompt(
        template="/prompts/author/generate-tests-from-oracle.md",
        input={
            "oracle": oracle,
            "language": "python",
            "framework": "pytest"
        }
    )
    print(f"Test code:\n{test_code}")
    
    # Human: Review tests
    if not human_approves(test_code):
        print("Tests rejected. Iterate.")
        return
    
    # Phase 3: Execute
    print("\nPhase 3: Executing tests...")
    execution_report = claude.prompt(
        template="/prompts/execute/run-tests-and-report.md",
        input={
            "test_code": test_code,
            "environment": environment,
            "mcp_allowlist": load_allowlist(feature_name),
            "oracle": oracle
        }
    )
    print(f"Execution report:\n{execution_report}")
    
    # Check for failures
    if "FAIL" in execution_report or "ERROR" in execution_report:
        # Phase 4: Heal
        print("\nPhase 4: Diagnosing failures...")
        diagnosis = claude.prompt(
            template="/prompts/heal/diagnose-test-failure.md",
            input={
                "failure": extract_failures(execution_report),
                "test_code": test_code,
                "oracle": oracle
            }
        )
        print(f"Diagnosis:\n{diagnosis}")
        
        # Human: Make fix decision
        if diagnosis.recommends("fix_product"):
            print("Product fix needed. Waiting for product team...")
            wait_for_product_fix()
            # Re-run Phase 3
            execution_report = claude.prompt(
                template="/prompts/execute/run-tests-and-report.md",
                input={...}
            )
    
    # Phase 5: Analyze
    print("\nPhase 5: Analyzing readiness...")
    analysis = claude.prompt(
        template="/prompts/analyze/summarize-results.md",
        input={
            "execution_report": execution_report,
            "oracle": oracle,
            "context": "new feature"
        }
    )
    print(f"Analysis:\n{analysis}")
    
    # Human: Final decision
    if analysis.ready_to_ship():
        print("✅ Ready to ship!")
        deploy(feature_name, environment)
    else:
        print("❌ Not ready. Fix blockers and retry.")
    
    return {
        "oracle": oracle,
        "test_code": test_code,
        "execution_report": execution_report,
        "analysis": analysis
    }
```

---

### Option C: Hybrid (Recommended)

Suitable for: Production use, balancing speed + control

**Time:** 60 minutes per feature (less back-and-forth)  
**Tokens:** 3-4k per feature  
**Cost:** ~$0.01 per feature

**Process:**

```
Human            Claude                      Product Team
─────────────────────────────────────────────────────────────
Read PRD
            → Phase 1: Extract oracle
                    ↓ (automated review)
            ← Oracle (auto-approved if matches PRD structure)
            
            → Phase 2: Generate tests
                    ↓ (automated review)
            ← Tests (auto-approved if covers oracle)
            
Set up MCP
allowlist
            → Phase 3: Execute tests
                    ↓ (automated, 1-5 min)
            ← Report (11/13 PASS)
            
Read report
Report shows          → Phase 4: Diagnose
bug in code     ← Diagnosis: "Fix endpoint to use generic message"
                
                                    Product team fixes
                                    ← Fix deployed
            
            → Phase 3: Re-execute (post-fix)
                    ↓ (automated)
            ← Report (13/13 PASS)
            
            → Phase 5: Analyze
                    ↓ (automated)
            ← Analysis: "Ready to ship"
            
Review
final report
            → Deploy
```

---

## MCP Allowlist Setup

Before running Phase 3 (Execute), define what endpoints the agent can access.

**Pattern:**

```yaml
# allowlist.yaml
endpoints:
  - pattern: https://api.staging.example.com/feature-endpoint
    methods: [GET, POST]
    description: "What this endpoint does"
    
  - pattern: https://api.staging.example.com/dependency
    methods: [GET]
    description: "Required dependency"

# Block by default:
# - production URLs
# - admin endpoints
# - endpoints with PII
```

**Validation:** Agent can only call endpoints in this list. Requests to any other endpoint are rejected before sending.

---

## Handling Failures

### If Phase 3 (Execute) Shows Failures

1. **Read the failure report carefully**
   - Which test failed?
   - What was expected?
   - What actually happened?

2. **Run Phase 4 (Heal)**
   - Ask: Is this a test bug, product bug, or oracle gap?
   - Get diagnosis + recommendation

3. **Take action based on diagnosis**
   - Test bug? Fix test, re-run Phase 3
   - Product bug? Product team fixes, re-run Phase 3
   - Oracle gap? Refine oracle, regenerate tests, re-run Phase 3

4. **Re-run Phase 3** after fix
   - If all pass: Continue to Phase 5 (Analyze)
   - If new failures: Return to Phase 4 (Heal)

---

## Success Criteria

**Phase 1 (Plan):** Oracle is explicit and complete  
→ Human can read it and say: "Yes, this is what matters"

**Phase 2 (Author):** Tests directly map to oracle  
→ Each test name matches an oracle claim

**Phase 3 (Execute):** All tests pass (or failures are diagnosed)  
→ Execution report clearly shows pass/fail + evidence

**Phase 4 (Heal):** Failures are understood  
→ Diagnosis explains root cause; recommendation is clear

**Phase 5 (Analyze):** Readiness is clear  
→ You can confidently decide: ship or don't ship

---

## Scaling This

### For 1 Feature
Use Option A (interactive). Takes 90 minutes. Worth the time to understand the flow.

### For 5-10 Features
Use Option B or C (scripted + human review). Batch them.
- Setup (MCP allowlists, environment): 30 min
- Per feature: 30-60 min
- Total: 3-6 hours for 5 features

### For 100+ Features
Use Option B (scripted) + automated approval gates.
- Setup CI/CD integration: 4 hours (one-time)
- Per feature: 30 min (mostly automated)
- Cost: ~$0.01 per feature

---

## Integration with Your Existing Process

**You have:** Manual testing, ad-hoc scripts, or Selenium tests  
**Problem:** Slow, expensive, incomplete coverage  
**Solution:** Adopt oracle-first workflow for new features; refactor old tests gradually

**Timeline:**
- Week 1: Implement workflow for 1 new feature (learn the process)
- Week 2-3: Implement for 3-5 features (refine your MCP allowlist)
- Week 4+: Scale up; integrate with CI/CD

---

## Prompts Reference

| Phase | Prompt | Purpose |
|-------|--------|---------|
| 1 | `/prompts/plan/extract-oracle-from-prd.md` | PRD → oracle |
| 2 | `/prompts/author/generate-tests-from-oracle.md` | Oracle → test code |
| 3 | `/prompts/execute/run-tests-and-report.md` | Tests → report |
| 4 | `/prompts/heal/diagnose-test-failure.md` | Failures → diagnosis |
| 5 | `/prompts/analyze/summarize-results.md` | Report → readiness |

---

## Troubleshooting

### Phase 1: Oracle is too vague
→ Clarify with product team or read PRD more carefully  
→ Re-run with more specific input

### Phase 2: Tests don't compile
→ Check syntax, environment (Python 3.9+?, pytest installed?)  
→ Regenerate tests with better error context

### Phase 3: Tests fail
→ Don't assume it's a product bug  
→ Run Phase 4 (Heal) to diagnose first

### Phase 3: Test times out
→ Increase timeout in MCP allowlist  
→ Check if API is actually slow or if test polling is wrong

### Phase 5: Can't decide if ready to ship
→ Look at coverage: are all critical paths tested?  
→ Look at risk: are security/data loss scenarios covered?  
→ Ask: "Would I be confident debugging this in production?"

---

## Links

- **Worked Example:** [sample-prd-walkthrough.md](sample-prd-walkthrough.md)
- **Prompts:** [/prompts/](../prompts/)
- **MCP Patterns:** [/mcp/](../mcp/)
- **Token Guide:** [/guides/token-tradeoffs.md](../guides/token-tradeoffs.md)

---

*Phase 3: Workflow orchestration. Ready to run.*


# NFR + Security Testing

Non-Functional Requirements: performance, load, security, chaos.

---

## What Is NFR Testing?

Functional: "Does password reset work?" ✓ (Phases 1-5)  
Non-Functional: "How fast? How many users? Is it secure?"

```
NFR Oracle (example):
  - Password reset must complete within 5 seconds (performance)
  - System must handle 1000 concurrent users (load)
  - No password should be logged (security)
  - API must be resilient to 50% packet loss (chaos)
```

---

## NFR Categories

### Performance Testing
Measure and assert on latency:

```python
def test_reset_response_time_sla():
    """Oracle: Response within 2 seconds (SLA)"""
    import time
    start = time.time()
    response = requests.post("/password-reset", json={"email": "test@example.com"})
    elapsed = time.time() - start
    
    assert elapsed < 2.0, f"SLA violated: {elapsed}s > 2s"
```

**Tools:** k6, Locust, Apache JMeter, custom timing assertions

### Load Testing
Verify system handles N concurrent users:

```python
# k6 example
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  vus: 100,  // 100 virtual users
  duration: '30s',
};

export default function() {
  let response = http.post('https://api.example.com/password-reset', {
    email: 'test@example.com',
  });
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 2s': (r) => r.timings.duration < 2000,
  });
}
```

### Security Testing
Validate against OWASP Top 10:

```python
def test_no_sql_injection():
    """Oracle: SQL injection attempts are blocked"""
    response = requests.post("/password-reset", json={
        "email": "' OR '1'='1"
    })
    
    # Should be treated as invalid email, not execute SQL
    assert response.status_code in [400, 200]  # 400 (invalid) or 200 (accepted)
    # Should NOT return user data or execute query

def test_no_xss_in_response():
    """Oracle: XSS payload is escaped"""
    response = requests.post("/password-reset", json={
        "email": "<script>alert('xss')</script>"
    })
    
    data = response.json()
    # Response should not contain raw script tag
    assert "<script>" not in json.dumps(data).lower()

def test_no_sensitive_data_in_logs():
    """Oracle: Passwords and tokens don't leak to logs"""
    # Check application logs
    logs = get_app_logs()
    
    assert "password" not in logs.lower()
    assert "token" not in logs.lower()
```

### Chaos Testing
Verify resilience to failures:

```python
def test_email_service_down():
    """Oracle: Graceful degradation if email service fails"""
    
    # Simulate email service being down
    with mock_service("email", status=500):
        response = requests.post("/password-reset", json={"email": "test@example.com"})
        
        # Should return graceful error, not crash
        assert response.status_code in [503, 500, 200]  # 503 Unavailable or async
        
        # Should not expose internal error details
        data = response.json()
        assert "java.lang.NullPointerException" not in str(data)
```

---

## NFR Test Patterns

### Pattern: Percentile Assertions

```python
def test_response_time_percentiles():
    """Oracle: 95th percentile <3s, 99th percentile <5s"""
    
    times = []
    for _ in range(100):
        start = time.time()
        requests.post("/password-reset", json={"email": f"test{_}@example.com"})
        times.append(time.time() - start)
    
    times.sort()
    p95 = times[95]  # 95th percentile
    p99 = times[99]  # 99th percentile
    
    assert p95 < 3.0, f"P95 {p95}s > 3s SLA"
    assert p99 < 5.0, f"P99 {p99}s > 5s SLA"
```

### Pattern: Error Rate Under Load

```python
def test_error_rate_under_load():
    """Oracle: <1% error rate at 100 concurrent users"""
    
    # Simulate 100 concurrent users
    results = []
    with ThreadPoolExecutor(max_workers=100) as executor:
        for i in range(100):
            future = executor.submit(
                requests.post,
                "/password-reset",
                json={"email": f"test{i}@example.com"}
            )
            results.append(future)
    
    responses = [r.result() for r in results]
    errors = sum(1 for r in responses if r.status_code >= 400)
    error_rate = errors / len(responses)
    
    assert error_rate < 0.01, f"Error rate {error_rate*100}% > 1%"
```

### Pattern: Resilience to Delays

```python
def test_resilience_to_slow_email_service():
    """Oracle: Works even if email takes 10+ seconds"""
    
    # Simulate slow email service (10 second delay)
    with mock_service_delay("email", delay_ms=10000):
        response = requests.post("/password-reset", json={"email": "test@example.com"})
        
        # Should still succeed (using async/queue internally)
        assert response.status_code == 200
```

---

## Phase 6 Scope

Phase 6 implements:
1. **Performance testing patterns** (k6 + assertions)
2. **Load testing harness** (concurrent user simulation)
3. **Security testing checklist** (OWASP Top 10)
4. **Chaos testing patterns** (service failures, delays)
5. **Gated NFR skill** (only run NFR tests when functional tests pass)

**Example:** A `nfr-skill` that:
- Checks Phase 5 tests all pass
- Runs performance suite (target SLA)
- Runs load suite (concurrent users)
- Runs security suite (injection, XSS, etc.)
- Reports readiness for high-traffic launch

---

## Integration with Phase 1-5

```
Phase 5 (Functional Testing) ✓ All pass
  ↓ Gate: Only proceed if all functional tests pass
Phase 6 (NFR Testing)
  ├─ Performance: Response time within SLA?
  ├─ Load: Handles N concurrent users?
  ├─ Security: Protected against OWASP Top 10?
  └─ Chaos: Resilient to failures?
      ↓ Gate: All NFR tests pass?
Phase 7 (Analyze)
  └─ Ready for high-traffic launch?
```

---

## Tools

| Tool | Best For | Language |
|------|----------|----------|
| **k6** | Load testing | JavaScript DSL |
| **Locust** | Python-based load testing | Python |
| **JMeter** | Load testing UI | Java |
| **OWASP ZAP** | Security scanning | Java |
| **Chaos Monkey** | Chaos engineering | Java |
| **Toxiproxy** | Network chaos | Go |

---

## Next Steps

1. Define NFR Oracle (SLA, load, security requirements)
2. Write NFR tests (performance, load, security)
3. Integrate with CI/CD (gate Phase 5 → Phase 6)
4. Monitor trends (performance regression detection)
5. Use in launch readiness assessment

---

## Links

- **k6 Docs:** https://k6.io/
- **OWASP Top 10:** https://owasp.org/www-project-top-ten/
- **Chaos Engineering:** https://principlesofchaos.org/
- **Phase 1-5 Workflow:** [/workflow/](../../workflow/)

---

*Phase 6: NFR testing. Ensure system performs, scales, and stays secure.*


# JUnit XML Format for Test Results

Standard format for test reports, compatible with CI/CD systems.

---

## Overview

JUnit XML is the de facto standard for test reporting. It's supported by:
- Jenkins, GitHub Actions, GitLab CI
- IDEs (VS Code, IntelliJ)
- Test dashboards (Allure, ReportPortal)
- Custom tools

---

## Schema

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuites name="password-reset-tests" tests="13" failures="2" skipped="0" time="42.3">
  <testsuite name="TestPasswordReset" tests="13" failures="2" skipped="0" time="42.3">
    
    <!-- PASS -->
    <testcase name="test_user_can_reset_password" classname="tests.test_reset.TestPasswordReset" time="0.85">
      <!-- No child elements = PASS -->
    </testcase>
    
    <!-- FAIL -->
    <testcase name="test_reset_email_arrives_within_30_seconds" classname="tests.test_reset.TestPasswordReset" time="30.2">
      <failure message="Timeout" type="AssertionError">
        Expected: email received within 30 seconds
        Actual: email did not arrive (timeout after 30s)
        
        Error:
          File "tests/test_reset.py", line 45, in test_reset_email_arrives_within_30_seconds
            assert email is not None
        AssertionError: Email not received
      </failure>
      <system-out>
        [DEBUG] Request: POST /password-reset
        [DEBUG] Response: 200 OK
        [DEBUG] Polled mailbox 30 times (timeout)
      </system-out>
      <system-err>
        [ERROR] Email service may be slow
      </system-err>
    </testcase>
    
    <!-- ERROR -->
    <testcase name="test_login_with_new_password" classname="tests.test_reset.TestPasswordReset" time="1.2">
      <error message="ConnectionError" type="requests.exceptions.ConnectionError">
        Failed to connect to API server
        
        Error:
          File "tests/test_reset.py", line 60, in test_login_with_new_password
            response = post("/login", data)
        requests.exceptions.ConnectionError: Max retries exceeded with url
      </error>
    </testcase>
    
    <!-- SKIP -->
    <testcase name="test_concurrent_resets" classname="tests.test_reset.TestPasswordReset" time="0">
      <skipped message="Concurrent reset testing planned for Phase 2">
        Reason: Out of scope for Phase 1
      </skipped>
    </testcase>
    
  </testsuite>
</testsuites>
```

---

## Key Elements

| Element | Purpose | Attributes |
|---------|---------|-----------|
| `<testsuites>` | Root container | `name`, `tests`, `failures`, `skipped`, `time`, `timestamp` |
| `<testsuite>` | Test class/module | `name`, `tests`, `failures`, `skipped`, `time`, `timestamp` |
| `<testcase>` | Individual test | `name`, `classname`, `time` |
| `<failure>` | Test failed | `message`, `type` |
| `<error>` | Test errored (exception) | `message`, `type` |
| `<skipped>` | Test skipped | `message` |
| `<system-out>` | Stdout | (text) |
| `<system-err>` | Stderr | (text) |

---

## Attributes Explained

### testsuites / testsuite

```xml
<testsuite 
  name="TestPasswordReset"           <!-- Test class/module name -->
  tests="13"                         <!-- Total test count -->
  failures="2"                       <!-- Failed test count -->
  errors="0"                         <!-- Errored test count -->
  skipped="0"                        <!-- Skipped test count -->
  time="42.3"                        <!-- Total execution time (seconds) -->
  timestamp="2025-06-10T15:30:00Z"   <!-- When test suite ran (ISO 8601) -->
  hostname="ci-runner-1"             <!-- Machine that ran tests (optional) -->
>
```

### testcase

```xml
<testcase 
  name="test_user_can_reset_password"              <!-- Test function name -->
  classname="tests.test_reset.TestPasswordReset"   <!-- Fully qualified class name -->
  time="0.85"                                       <!-- Execution time (seconds) -->
  file="tests/test_reset.py"                        <!-- Source file (optional) -->
  line="25"                                         <!-- Line number (optional) -->
>
```

### failure / error

```xml
<failure 
  message="AssertionError: Email not received"     <!-- Short error message -->
  type="AssertionError"                            <!-- Exception type -->
>
Expected: email received within 30 seconds
Actual: email did not arrive

Full traceback and context...
</failure>
```

---

## Complete Example: Password Reset

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuites name="password-reset-tests" tests="13" failures="1" skipped="1" time="45.7" timestamp="2025-06-10T15:30:00Z">
  
  <testsuite name="TestPasswordReset" tests="13" failures="1" skipped="1" time="45.7">
    
    <!-- Happy Path Tests -->
    
    <testcase name="test_user_can_request_reset" classname="tests.test_reset.TestPasswordReset" time="0.85" />
    
    <testcase name="test_reset_email_arrives_within_30_seconds" classname="tests.test_reset.TestPasswordReset" time="2.3">
      <failure message="Timeout: email did not arrive" type="AssertionError">
Expected: email in mailbox within 30 seconds
Actual: timeout after 30 seconds

Diagnostic:
  - Request: POST /password-reset with test@example.com
  - Response: 200 OK
  - Mailbox polling: 30 attempts, 1s interval
  - Result: no email found

Likely causes:
  - Email service is slow
  - Test timeout is too tight (SLA is 30s, test waits 30s but misses delivery)
  - Mailbox API is flaky

Traceback:
  File "tests/test_reset.py", line 45
    assert email is not None
AssertionError
      </failure>
      <system-out>
[15:30:02.100] POST /password-reset {"email": "test@example.com"}
[15:30:02.150] Response: 200 OK
[15:30:02.200] Mailbox poll attempt 1/30: no email (retrying in 1s)
[15:30:03.200] Mailbox poll attempt 2/30: no email (retrying in 1s)
...
[15:30:32.200] Mailbox poll attempt 30/30: no email (timeout)
      </system-out>
    </testcase>
    
    <testcase name="test_reset_link_is_valid" classname="tests.test_reset.TestPasswordReset" time="1.2" />
    
    <testcase name="test_user_can_set_new_password" classname="tests.test_reset.TestPasswordReset" time="0.95" />
    
    <testcase name="test_user_can_login_with_new_password" classname="tests.test_reset.TestPasswordReset" time="1.1" />
    
    <testcase name="test_user_logged_out_after_reset" classname="tests.test_reset.TestPasswordReset" time="0.88" />
    
    <!-- Edge Cases -->
    
    <testcase name="test_reset_link_expires_after_24_hours" classname="tests.test_reset.TestPasswordReset" time="0.7" />
    
    <!-- Error Conditions -->
    
    <testcase name="test_empty_email_returns_error" classname="tests.test_reset.TestPasswordReset" time="0.6" />
    
    <testcase name="test_malformed_email_returns_error" classname="tests.test_reset.TestPasswordReset" time="0.65" />
    
    <testcase name="test_nonexistent_email_returns_generic_error" classname="tests.test_reset.TestPasswordReset" time="0.7" />
    
    <testcase name="test_expired_link_returns_error" classname="tests.test_reset.TestPasswordReset" time="0.75" />
    
    <!-- Additional Tests -->
    
    <testcase name="test_rate_limiting" classname="tests.test_reset.TestPasswordReset" time="0">
      <skipped message="Out of scope for Phase 1; planned for Phase 2">
        Rate limiting is handled at API gateway level; no dedicated test needed yet
      </skipped>
    </testcase>
    
    <testcase name="test_concurrent_resets" classname="tests.test_reset.TestPasswordReset" time="0">
      <skipped message="Concurrent reset testing planned for Phase 2">
        Concurrent behavior not specified in Phase 1 oracle
      </skipped>
    </testcase>
    
  </testsuite>
  
</testsuites>
```

---

## Generating JUnit XML

### From Pytest

```bash
# Auto-generate JUnit XML
pytest tests/test_reset.py -v --junit-xml=results.xml

# With additional options
pytest tests/test_reset.py -v \
  --junit-xml=results.xml \
  --junit-xml-prefix=PASS \
  --tb=short
```

### From Other Frameworks

```bash
# Jest
npm test -- --reporters=default --reporters=jest-junit

# TestNG (Java)
mvn test -DargLine="-Dorg.uncommons.reportng.output-dir=results"

# Other: implement custom reporter or use tool
```

---

## Parsing & Using JUnit XML

### Extract Pass Rate

```python
import xml.etree.ElementTree as ET

def extract_metrics(junit_file):
    tree = ET.parse(junit_file)
    root = tree.getroot()
    
    total = int(root.get("tests", 0))
    failures = int(root.get("failures", 0))
    errors = int(root.get("errors", 0))
    skipped = int(root.get("skipped", 0))
    
    passed = total - failures - errors - skipped
    pass_rate = (passed / total * 100) if total > 0 else 0
    
    return {
        "total": total,
        "passed": passed,
        "failed": failures,
        "errors": errors,
        "skipped": skipped,
        "pass_rate": pass_rate
    }

# Usage:
metrics = extract_metrics("results.xml")
print(f"Pass rate: {metrics['pass_rate']:.1f}%")
```

### Extract Flaky Tests

```python
def extract_flaky_tests(junit_files):
    """Identify flaky tests from multiple runs"""
    
    test_results = {}
    
    for junit_file in junit_files:
        tree = ET.parse(junit_file)
        root = tree.getroot()
        
        for testcase in root.iter("testcase"):
            test_name = testcase.get("name")
            is_pass = len(testcase) == 0  # No failure/error/skipped child
            
            if test_name not in test_results:
                test_results[test_name] = []
            test_results[test_name].append(is_pass)
    
    # Find flaky (not 100% pass or 0% pass)
    flaky = {}
    for test_name, results in test_results.items():
        pass_count = sum(results)
        total = len(results)
        pass_rate = pass_count / total
        
        if 0 < pass_rate < 1.0:  # Some pass, some fail
            flaky[test_name] = pass_rate
    
    return flaky

# Usage:
flaky = extract_flaky_tests(["run1.xml", "run2.xml", "run3.xml"])
for test_name, pass_rate in flaky.items():
    print(f"{test_name}: {pass_rate*100:.1f}% pass")
```

---

## CI/CD Integration

### GitHub Actions

```yaml
- name: Run tests
  run: pytest tests/ --junit-xml=results.xml

- name: Publish test results
  if: always()
  uses: EnricoMi/publish-unit-test-result-action@v2
  with:
    files: results.xml
    check_name: Password Reset Tests
```

### Jenkins

```groovy
stage('Test') {
    steps {
        sh 'pytest tests/ --junit-xml=results.xml'
    }
    post {
        always {
            junit 'results.xml'
        }
    }
}
```

### GitLab CI

```yaml
test:
  script:
    - pytest tests/ --junit-xml=results.xml
  artifacts:
    reports:
      junit: results.xml
```

---

## Best Practices

1. **Always include timestamps** — enables trend analysis
2. **Include full errors** — copy assertion message + traceback
3. **Add system-out/system-err** — logs help debugging
4. **Categorize tests** — use classname to group related tests
5. **Record execution time** — identify slow tests
6. **Use consistent naming** — makes parsing and reporting easier

---

## Links

- **Allure Integration:** [allure-integration.md](allure-integration.md)
- **Phase 3 (Execute):** [/prompts/execute/run-tests-and-report.md](../prompts/execute/run-tests-and-report.md)
- **Flaky Detection:** [/self-healing/flaky-test-detection.md](../self-healing/flaky-test-detection.md)

---

*Phase 4: JUnit XML reporting. Structure your test data.*


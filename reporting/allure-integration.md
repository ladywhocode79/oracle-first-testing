# Allure Report Integration

Rich, visual test reporting with trends, categorization, and history.

---

## What Is Allure?

Allure is a test reporting framework that transforms raw test results into:

- **Dashboard:** Pass rate, failures, flakiness at a glance
- **Trends:** How are tests trending over time?
- **Categories:** Group tests by feature, severity, priority
- **History:** Which tests fail consistently? Which are flaky?
- **Details:** Each test has logs, attachments, timing, environment

---

## Allure Report Structure

```
Allure Report/
├── Dashboard
│   ├── Overview (pass rate, duration, flakiness)
│   ├── Categories (by feature, by priority, by severity)
│   └── Trends (pass rate over time, failure rate, flakiness)
├── Tests
│   ├── test_user_can_reset_password (PASS, 0.85s)
│   │   └── Details: logs, duration, environment, history
│   ├── test_reset_email_arrives (FAIL, 2.3s)
│   │   ├── Failure reason
│   │   ├── Error message
│   │   ├── Logs
│   │   └── History (how often does it fail?)
│   └── ...
├── Suites
│   ├── TestPasswordReset (13 tests, 11 pass, 2 fail)
│   └── ...
└── Behavior
    ├── Feature: Password Reset
    │   ├── Epic: Authentication
    │   └── Story: User can reset forgotten password
    └── ...
```

---

## Setup

### 1. Install Allure

```bash
# MacOS
brew install allure

# Linux
sudo apt-add-repository ppa:qameta/allure
sudo apt-get update
sudo apt-get install allure

# Or download from: https://github.com/allure-framework/allure2/releases
```

### 2. Install Test Adapter

```bash
# For pytest
pip install pytest allure-pytest

# For Jest
npm install --save-dev jest allure-jest

# For TestNG (Java)
mvn dependency:get -Dartifact=io.qameta.allure:allure-testng:2.20.1
```

### 3. Generate Allure Results

```bash
# Pytest: generate results directory
pytest tests/test_reset.py -v --alluredir=allure-results

# Jest: generate results
jest --reporters=default --reporters=jest-allure
```

### 4. Generate Report

```bash
# Generate HTML report
allure generate allure-results -o allure-report

# Serve report locally
allure serve allure-results
```

---

## Allure Annotations (pytest)

Annotate tests to enrich Allure report:

```python
import allure
from allure import severity, severity_level

class TestPasswordReset:
    
    @allure.title("User Can Request Password Reset")
    @allure.description("Verify user can request a password reset by email")
    @severity(severity_level.CRITICAL)
    @allure.feature("Password Reset")
    @allure.story("User requests reset")
    @allure.tag("authentication", "happy-path")
    def test_user_can_request_reset(self):
        """Oracle: User can request password reset"""
        with allure.step("POST /password-reset"):
            response = post("/password-reset", {"email": "test@example.com"})
        
        with allure.step("Assert status 200"):
            assert response.status_code == 200
        
        with allure.step("Assert success message"):
            assert "reset" in response.json()["message"].lower()
    
    @allure.title("Reset Email Arrives Within 30 Seconds")
    @severity(severity_level.CRITICAL)
    @allure.feature("Password Reset")
    @allure.story("Email delivery")
    @allure.tag("email", "sla")
    def test_reset_email_arrives(self):
        """Oracle: Email arrives within 30 seconds"""
        with allure.step("Request password reset"):
            response = post("/password-reset", {"email": "test@example.com"})
        
        with allure.step("Poll mailbox for 30 seconds"):
            start = time.time()
            email = None
            while time.time() - start < 30:
                email = get_mailbox()
                if email:
                    break
                time.sleep(0.5)
        
        elapsed = time.time() - start
        
        with allure.step(f"Assert email arrived (took {elapsed:.1f}s)"):
            assert email is not None
            assert elapsed < 30
        
        # Attach email for inspection
        with allure.step("Attach email"):
            allure.attach(
                email.subject,
                name="email_subject",
                attachment_type=allure.attachment_type.TEXT
            )
    
    @allure.title("Nonexistent Email Returns Generic Error")
    @severity(severity_level.CRITICAL)
    @allure.feature("Password Reset")
    @allure.tag("security", "user-enumeration")
    def test_nonexistent_email_generic_error(self):
        """Oracle: Don't reveal whether email exists"""
        with allure.step("POST /password-reset with fake email"):
            response = post("/password-reset", {"email": "fake@example.com"})
        
        with allure.step("Assert status 200"):
            assert response.status_code == 200
        
        with allure.step("Assert message is generic (no 'not found')"):
            message = response.json()["message"]
            assert "not found" not in message.lower()
            assert "registered" not in message.lower()
            allure.attach(
                f"Message: {message}",
                name="response_message",
                attachment_type=allure.attachment_type.TEXT
            )
```

---

## Allure Categories

Group tests by feature, severity, or layer:

```python
# Define categories in conftest.py
import allure

def pytest_configure(config):
    """Configure Allure categories"""
    # This tells Allure how to group tests
```

Create `allure-categories.json`:

```json
[
  {
    "name": "Happy Path",
    "matchedStatuses": ["passed"],
    "matchedStatuses": ["passed"]
  },
  {
    "name": "Flaky",
    "matchedStatuses": ["passed", "failed"],
    "flaky": true
  },
  {
    "name": "Security",
    "matchedStatuses": ["failed"],
    "matchedStatuses": ["failed"]
  },
  {
    "name": "Product Bugs",
    "matchedStatuses": ["failed"]
  }
]
```

---

## Allure in CI/CD

### GitHub Actions

```yaml
- name: Run tests with Allure
  run: pytest tests/ --alluredir=allure-results

- name: Generate Allure report
  if: always()
  run: allure generate allure-results -o allure-report

- name: Deploy Allure report
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: allure-report
    path: allure-report
```

### GitHub Pages

```yaml
- name: Deploy Allure to GitHub Pages
  if: always()
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./allure-report
```

Then view at: `https://yourname.github.io/repo/`

---

## Allure + Flaky Detection

### Add Flaky Tests List

Create `allure-flaky.json`:

```json
[
  {
    "testId": "test_reset_email_arrives",
    "flakyReason": "Email service in staging is slow (SLA 30s, sometimes takes 15-20s)"
  },
  {
    "testId": "test_concurrent_resets",
    "flakyReason": "Race condition with shared state; isolated tests are 100% reliable"
  }
]
```

This will be reflected in Allure dashboard.

---

## Allure Trends

Allure automatically tracks trends over time:

```
Pass Rate Trend:
  Jun 1:  100% (13/13)
  Jun 8:  92%  (12/13) ← degradation detected
  Jun 15: 85%  (11/13) ← worsening
  Jun 22: 100% (13/13) ← fixed!

Flakiness Trend:
  Jun 1:  0%
  Jun 8:  7%  (1 test flaky)
  Jun 15: 7%
  Jun 22: 0%  (flaky test fixed)

Duration Trend:
  Avg: 3.3s
  Min: 2.8s
  Max: 4.1s
```

---

## Example: Allure Report View

### Dashboard Tab

```
┌─────────────────────────────────────────────────────┐
│ Password Reset Tests                                │
├─────────────────────────────────────────────────────┤
│ Status Overview:                                    │
│   ✓ Passed: 11 (85%)  [████████░░]                 │
│   ✗ Failed: 1  (8%)                                │
│   ⚠ Flaky:  1  (8%)                                │
│   ⊝ Skipped: 0                                      │
│                                                     │
│ Duration: 45.7 seconds                              │
│ Environment: staging                                │
│                                                     │
│ By Feature:                                         │
│   Password Reset: 11/13 (85%)                       │
│   Email Delivery: 1/2 (50%) ← flaky                │
│   Error Handling: 5/5 (100%)                        │
│                                                     │
│ By Severity:                                        │
│   Critical: 9/11 PASS (1 failed, 1 flaky)          │
│   High: 4/4 PASS                                    │
│   Medium: 0/0 N/A                                   │
└─────────────────────────────────────────────────────┘
```

### Tests Tab

```
┌─────────────────────────────────────────────────────────┐
│ test_user_can_reset_password          ✓ PASS  0.85s    │
│ test_reset_email_arrives              ✗ FAIL  30.2s   │
│ test_reset_link_is_valid              ✓ PASS  1.2s    │
│ test_user_can_login_with_new_password ✓ PASS  1.1s    │
│ test_nonexistent_email_generic_error  ✓ PASS  0.7s    │
│ ...                                                     │
└─────────────────────────────────────────────────────────┘

Click on any test to see:
  - Full error message
  - Stack trace
  - Logs (system-out, system-err)
  - Duration
  - Attachments (screenshots, emails)
  - History (how often does it fail?)
```

### Trends Tab

```
Pass Rate Trend:
  100% ├─────────────────────┐
       │                     │
   95% │                     ├──┐
       │                     │  │
   90% │                     │  ├──
       │                     │  │
   Jun 1   Jun 8   Jun 15   Jun 22

Failure Categories:
  Flaky: 1
  Email: 1
  Product Bug: 0
```

---

## Best Practices

1. **Annotate all tests** — title, description, severity, tags
2. **Use steps** — break tests into logical steps for better reporting
3. **Attach artifacts** — logs, screenshots, response bodies
4. **Set severity** — critical vs. high vs. medium for filtering
5. **Tag consistently** — use same tags across tests for grouping
6. **Review trends** — weekly, look for regressions
7. **Link to issues** — "@link https://github.com/issue/123"

---

## Integration with Phase 4

1. **Phase 3 (Execute):** Generates raw test results
2. **Allure:** Transforms into rich report with trends/analysis
3. **Phase 4 (Heal):** Uses Allure to identify flaky tests
4. **Monitoring:** Track trends, detect regressions early

---

## Links

- **JUnit XML:** [junit-xml-format.md](junit-xml-format.md)
- **Flaky Detection:** [/self-healing/flaky-test-detection.md](../self-healing/flaky-test-detection.md)
- **Allure Official:** https://docs.qameta.io/allure/

---

*Phase 4: Allure reporting. Visualize test health and trends.*


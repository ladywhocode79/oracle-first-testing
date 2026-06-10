# Appium Track: Mobile Testing

Test iOS and Android apps using the oracle-first approach.

---

## What Is Appium?

Appium is a tool for automating mobile apps (iOS, Android, Windows). It uses WebDriver protocol (same as Selenium), making it familiar to web testers.

**Key points:**
- Works with native apps, web apps, hybrid apps
- Uses standard WebDriver commands (click, type, assert)
- Supports both iOS (XCUITest) and Android (UIAutomator2)
- Can run on real devices or simulators

---

## Oracle for Mobile

Mobile-specific test claims:

```
# Password Reset Oracle (Mobile)

## Happy Path (Critical)
- User can tap "Forgot Password?" button
- Reset screen is displayed (wait for element, not page load)
- User can type email into text field
- User can tap "Send Reset Link" button
- Success message appears (toast or alert)
- Email with reset link arrives within 30 seconds

## Touch Interactions (Mobile-specific)
- Swipe gestures work (if needed)
- Keyboard appears/dismisses correctly
- Orientation changes don't break the flow (portrait ↔ landscape)
- Text field focus/blur works

## Error Conditions
- Invalid email shows validation error (inline, not page navigation)
- Network error shows appropriate message
- Back button returns to login screen
- App doesn't crash under stress

## Native Elements
- Uses native input fields (not web inputs)
- Alerts use native OS dialogs
- Navigation uses native back button
```

---

## Appium MCP Pattern

```python
# Tools exposed via Appium MCP

Tool: appium/find_element
Input: { locator: "id|xpath|text|classname", value: string }
Output: { element_id: string, visible: bool }

Tool: appium/click
Input: { element_id: string }
Output: { status: "success" | "failed" }

Tool: appium/type
Input: { element_id: string, text: string }
Output: { status: "success" | "failed" }

Tool: appium/assert_text
Input: { element_id: string, expected_text: string }
Output: { pass: bool, actual_text: string }

Tool: appium/get_screenshot
Input: {}
Output: { image_path: string }

Tool: appium/tap
Input: { x: number, y: number }
Output: { status: "success" }

Tool: appium/swipe
Input: { from_x: number, from_y: number, to_x: number, to_y: number }
Output: { status: "success" }

Tool: appium/wait_for_element
Input: { locator: string, value: string, timeout: number }
Output: { found: bool, element_id: string }
```

---

## Test Example: Password Reset (Mobile)

```python
# Using Appium + oracle-first approach

class TestPasswordResetMobile:
    """Password reset on mobile app (iOS/Android)"""
    
    def setup_method(self):
        """Launch app, navigate to login"""
        self.driver = appium_connect("ios")  # or "android"
        # App launches to login screen
    
    def test_user_can_request_reset(self):
        """Oracle: User taps Forgot Password, enters email, submits"""
        
        # Find and tap "Forgot Password?" button
        forgot_btn = self.driver.find_element_by_id("forgot_password_button")
        forgot_btn.click()
        
        # Wait for reset screen to appear
        reset_header = self.driver.wait_for_element(
            locator="text",
            value="Reset Password",
            timeout=5
        )
        assert reset_header is not None
        
        # Find email input field (note: native element, not web input)
        email_field = self.driver.find_element_by_id("reset_email_input")
        email_field.send_keys("test@example.com")
        
        # Tap "Send Reset Link" button
        send_btn = self.driver.find_element_by_id("send_reset_link_button")
        send_btn.click()
        
        # Wait for success message (toast notification)
        success_msg = self.driver.wait_for_element(
            locator="text",
            value="Reset link sent",
            timeout=5
        )
        assert success_msg is not None
    
    def test_invalid_email_shows_error(self):
        """Oracle: Invalid email shows validation error"""
        
        # Navigate to reset screen
        self.driver.find_element_by_id("forgot_password_button").click()
        self.driver.wait_for_element(locator="text", value="Reset Password")
        
        # Enter invalid email
        email_field = self.driver.find_element_by_id("reset_email_input")
        email_field.send_keys("not-an-email")
        
        # Tap submit
        self.driver.find_element_by_id("send_reset_link_button").click()
        
        # Wait for error message (inline validation)
        error_msg = self.driver.wait_for_element(
            locator="id",
            value="email_error_message",
            timeout=2
        )
        assert error_msg is not None
        assert "invalid" in error_msg.text.lower()
    
    def test_orientation_change_preserves_state(self):
        """Oracle: Portrait ↔ landscape doesn't lose form data"""
        
        # Navigate to reset, enter email
        self.driver.find_element_by_id("forgot_password_button").click()
        email_field = self.driver.find_element_by_id("reset_email_input")
        email_field.send_keys("test@example.com")
        
        # Rotate device (portrait to landscape)
        self.driver.orientation = "landscape"
        
        # Verify email is still there (not cleared)
        email_field = self.driver.find_element_by_id("reset_email_input")
        assert email_field.get_attribute("value") == "test@example.com"
        
        # Rotate back
        self.driver.orientation = "portrait"
        
        # Still there
        email_field = self.driver.find_element_by_id("reset_email_input")
        assert email_field.get_attribute("value") == "test@example.com"
```

---

## Setup & Configuration

### Prerequisites

```bash
# Install Appium
npm install -g appium

# Install drivers (once per setup)
appium driver install xcuitest   # iOS
appium driver install uiautomator2  # Android

# Install Python client
pip install appium-python-client
```

### Start Appium Server

```bash
# Terminal 1: Start Appium server
appium

# Should output:
# Appium v2.x.x running on http://localhost:4723
```

### Configure Test Environment

```yaml
# appium-config.yaml
capabilities:
  ios:
    platformName: iOS
    platformVersion: "17.0"
    deviceName: "iPhone 15 Simulator"
    app: "/path/to/app.app"
    automationName: "XCUITest"
  
  android:
    platformName: Android
    platformVersion: "14"
    deviceName: "emulator-5554"
    app: "/path/to/app.apk"
    automationName: "UIAutomator2"
    appActivity: "com.example.MainActivity"
```

---

## Common Locators

| Type | XPath Example | Note |
|------|---|---|
| **ID** | `id("reset_email_input")` | Fastest, most reliable |
| **Text** | `text("Send Reset Link")` | For button labels |
| **Class** | `classname("android.widget.EditText")` | For input fields |
| **Predicate (iOS)** | `predicate("name contains 'reset'"` | iOS-specific |
| **UiSelector (Android)** | `new UiSelector().text("Send")` | Android-specific |

**Best practice:** Use IDs whenever possible (most stable).

---

## Mobile-Specific Challenges

### Challenge 1: Slow Element Rendering

Mobile apps can be slower than web apps. Always use waits:

```python
# ✗ Don't: Assume element appears immediately
element = self.driver.find_element_by_id("email_field")

# ✓ Do: Wait for element
element = self.driver.wait_for_element(
    locator="id",
    value="email_field",
    timeout=5
)
```

### Challenge 2: Keyboard Behavior

Keyboard appears/disappears, sometimes covering fields:

```python
# Close keyboard after typing
self.driver.hide_keyboard()

# Or swipe up to reveal covered element
self.driver.swipe(
    start_x=400, start_y=600,
    end_x=400, end_y=200
)
```

### Challenge 3: Orientation & Screen Size

Mobile apps can rotate; test data might need adjustment:

```python
def test_works_portrait_and_landscape(self):
    # Test in portrait
    assert self.submit_button.is_displayed()
    
    # Rotate to landscape
    self.driver.orientation = "landscape"
    
    # Button might be in different position, but should still be accessible
    assert self.submit_button.is_displayed()
```

### Challenge 4: Network Conditions

Test apps under slow network:

```python
# Simulate slow network (Android only via Appium)
self.driver.set_network_connection(
    "wifi_and_data"  # All on
    # or "wifi_only"
    # or "airplane_mode"
)
```

---

## Integration with Phase 1-4

**Phase 1 (Plan):** Extract mobile-specific oracle (touch, native elements, orientation)

**Phase 2 (Author):** Generate Appium test code from oracle

**Phase 3 (Execute):** Run Appium tests via Appium MCP

**Phase 4 (Heal):** Diagnose flaky tests (timing? keyboard? network?)

**Phase 5 (Analyze):** Same readiness assessment

---

## Appium vs. Playwright for Mobile

| Aspect | Appium | Playwright (mobile) |
|--------|--------|---|
| **Native apps** | ✓ Yes | ✗ No (web only) |
| **Hybrid apps** | ✓ Yes | ✓ Yes |
| **iOS** | ✓ Yes | ✓ Yes (chromium) |
| **Android** | ✓ Yes | ✓ Yes (chromium) |
| **Token cost** | Medium (device launch) | Low (emulated) |
| **Maturity** | Mature | Newer, improving |
| **Use case** | Native app testing | Mobile web testing |

**Recommendation:**
- Native app → Appium
- Mobile web → Playwright
- Hybrid → Either (Appium for native features)

---

## Example: Running Password Reset on iOS

```bash
# 1. Start Appium
appium

# 2. Run tests
pytest tests/test_password_reset_mobile.py -v \
  --appium-platform iOS \
  --appium-device "iPhone 15 Simulator"

# 3. View results
allure serve allure-results
```

Results include:
- Pass/fail for each interaction
- Screenshots (tap locations, errors)
- Network timing
- Device logs (errors, warnings)

---

## Next Steps

1. **Read `oracle-adaptation.md`** — Mobile-specific oracle claims
2. **Follow `examples/password-reset/`** — Worked example (iOS + Android)
3. **Integrate with workflow** — Use same Phase 1-5 pipeline
4. **Scale** — Add tests for other mobile features

---

## Links

- **Oracle Adaptation:** [oracle-adaptation.md](oracle-adaptation.md)
- **Worked Example:** [examples/password-reset/](examples/password-reset/)
- **Appium Docs:** https://appium.io/docs/
- **Appium Python Client:** https://github.com/appium/python-client
- **Phase 1-4 Workflow:** [/workflow/](../../workflow/)

---

*Phase 5: Mobile testing with Appium. Same oracle-first approach, different platform.*


# Test Automation Tracks

Extending oracle-first testing to multiple platforms and technologies.

Each track applies the same doctrine (oracle-executor split) to a different domain.

---

## Available Tracks

### Core Tracks (Phase 1-4)
- **REST API Testing** (CLI MCP) — via `/mcp/cli-mcp-pattern.md`
- **Web UI Testing** (Playwright MCP) — via `/mcp/playwright-mcp-pattern.md`

### Modernized Tracks (Phase 5)

#### Track: Mobile Testing (Appium)
- **Appium MCP Pattern** — Drive iOS/Android apps via Appium
- **Oracle Adaptation** — Mobile-specific test claims (tap, swipe, native elements)
- **Examples** — Password reset on mobile app
- **Integration** — Works with existing Phase 1-4 workflow

#### Track: Contract Testing
- **Contract Oracle** — API schemas, request/response validation
- **Pact/OpenAPI Integration** — Validate client ↔ server contracts
- **Examples** — Password reset API contract tests
- **Integration** — Phase alongside API tests, before integration tests

#### Track: Playwright for Java
- **Why:** Java teams want Playwright (modern, fast) instead of Selenium
- **Integration** — Drop-in replacement for Selenium in TestNG suites
- **Examples** — Same password reset test in Java + Playwright

#### Track: Synthetic Data Generation
- **Faker/Factory Boy patterns** — Generate realistic test data
- **Test Data Oracles** — "What makes valid test data?"
- **Examples** — Synthetic users, emails, passwords
- **Integration** — Feeds into all test types

#### Track: Visual Testing
- **Image Assertions** — Screenshot comparison, visual regression
- **Tools** — Applitools, Percy, or custom image diffing
- **Examples** — Password reset form layout consistency
- **Integration** — Complements functional assertions

---

## Track Structure

Each track follows this pattern:

```
/tracks/[track-name]/
├── README.md                  (Overview)
├── mcp-pattern.md            (If applicable: MCP tool definitions)
├── oracle-adaptation.md       (Domain-specific oracle claims)
├── examples/
│   ├── password-reset/       (Worked example in this track)
│   │   ├── oracle.md         (Oracle adapted for this track)
│   │   ├── test-code.md      (Test implementation)
│   │   └── execution.md       (Real execution results)
│   └── ...
└── integration-guide.md       (How to integrate with Phase 1-4 workflow)
```

---

## Which Track to Use When?

| What You're Testing | Track | Why |
|---|---|---|
| REST/GraphQL API | REST API (CLI MCP) | Fast, cheap, simple |
| Web UI | Web UI (Playwright MCP) | Only way to test UI |
| Mobile app | Appium | Native app testing, touch events |
| API contract | Contract Testing | Prevent client/server desync |
| Java backend + UI | Playwright for Java | Modern, faster than Selenium |
| Test data setup | Synthetic Data | Realistic, repeatable data |
| Visual regressions | Visual Testing | Catch layout/style changes |

---

## Phase 5 Scope

Phase 5 implements:
1. **Appium MCP Pattern** (mobile testing)
2. **Contract Testing Framework** (API schemas)
3. **Synthetic Data Patterns** (test data generation)
4. **Visual Testing Integration** (screenshot comparison)
5. Worked examples for each

Not in Phase 5 (future):
- Playwright for Java (straightforward, lower priority)
- Performance testing details (covered in Phase 6)
- Security scanning (covered in Phase 6)

---

## Integration with Existing Phases

```
Phase 1 (Plan) → Extract oracle
  └─ [Track-specific] Adapt oracle to domain (mobile? contract? visual?)

Phase 2 (Author) → Generate tests
  └─ [Track-specific] Use track-specific syntax (Appium? Pact? Faker?)

Phase 3 (Execute) → Run tests
  └─ [Track-specific] Use track-specific MCP or tool

Phase 4 (Heal) → Diagnose
  └─ [Track-agnostic] Patterns apply to all tracks

Phase 5 (Analyze) → Readiness
  └─ [Track-agnostic] Same assessment logic
```

All tracks feed into unified reporting (JUnit + Allure).

---

## Getting Started

Pick a track based on what you're testing:
1. Read the track's `README.md`
2. Read the `oracle-adaptation.md` (understand domain-specific claims)
3. Follow the worked example in `examples/password-reset/`
4. Integrate with your Phase 1-4 workflow

---

## Links

- **Appium Track:** [appium/README.md](appium/README.md)
- **Contract Testing Track:** [contract-testing/README.md](contract-testing/README.md)
- **Synthetic Data Track:** [synthetic-data/README.md](synthetic-data/README.md)
- **Visual Testing Track:** [visual-testing/README.md](visual-testing/README.md)

---

*Phase 5: Extend oracle-first to multiple domains. Same doctrine, different domains.*


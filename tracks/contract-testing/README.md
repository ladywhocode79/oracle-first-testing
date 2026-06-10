# Contract Testing Track

Validate API contracts between client and server: request/response schemas, formats, required fields.

---

## What Is Contract Testing?

A contract is an agreement: "This API endpoint will always return this structure."

```
Server promise: "I'll return { message: string, resetUrl: string }"
Client assumption: "I expect { message: string, resetUrl: string }"

Contract test: Verify the server keeps its promise ✓
```

Without contract testing:
- Server changes API → Client breaks (no warning)
- Client assumes field X is always present → Null exception in production
- One team deploys breaking change → Other team's tests fail

With contract testing:
- Changes are caught before deploy
- Client and server are always in sync
- Contracts serve as living documentation

---

## Contract Oracle

Oracle for API contracts:

```
# Password Reset API Contract

## POST /password-reset

Request:
  - email: string (required)
  - format: JSON

Response (200):
  - message: string (required) - human-readable confirmation
  - resetUrl?: string (optional) - link to reset form
  - expiresAt?: string (optional, ISO 8601) - token expiry time

Response (400):
  - error: string (required) - error message

Response (401):
  - message: string (required) - generic message
  
No field in response should expose sensitive data (token, user ID, etc.)

## POST /reset

Request:
  - token: string (required)
  - newPassword: string (required)
  - format: JSON

Response (200):
  - message: string (required) - "Password reset successfully"
  - redirectUrl?: string (optional) - where to send user

Response (401):
  - message: string (required) - "Token expired or invalid"

Contract: All responses use ISO 8601 for dates, no arbitrary formats
```

---

## Tools for Contract Testing

### Option 1: Pact (Best for Microservices)

Pact records client interactions, verifies server provides them:

```python
from pact import Consumer, Provider

# Client test: define what you expect
pact = Consumer("PasswordResetClient").has_state("user exists").upon_receiving(
    "a request to reset password"
).with_request("post", "/password-reset", body={"email": "test@example.com"}).will_respond_with(
    200, body={"message": "reset link sent"}
)

with pact:
    response = requests.post("http://localhost:8000/password-reset", json={"email": "test@example.com"})
    assert response.json()["message"] == "reset link sent"

# Pact file is generated: pact/PasswordResetClient-PasswordResetServer.json

# Server test: verify it provides what's promised
provider = Provider("PasswordResetServer")
provider.given("user exists").upon_receiving(
    "a request to reset password"
).with_request("post", "/password-reset", body={"email": "test@example.com"}).will_respond_with(
    200, body={"message": "reset link sent"}
)
provider.verify()  # Verifies actual server matches pact file
```

**Best for:** Microservices, independent client/server teams

### Option 2: OpenAPI + Schema Validation (Best for REST)

Define API contract in OpenAPI, validate responses:

```yaml
# openapi.yaml
openapi: 3.0.0
paths:
  /password-reset:
    post:
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email:
                  type: string
                  format: email
              required:
                - email
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  message:
                    type: string
                  resetUrl:
                    type: string
                    format: uri
                required:
                  - message
        '400':
          description: Bad request
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
                required:
                  - error
```

Test:

```python
from openapi_spec_validator import validate_spec
import yaml

# Validate server conforms to OpenAPI spec
with open("openapi.yaml") as f:
    spec = yaml.safe_load(f)

response = requests.post("http://localhost:8000/password-reset", json={"email": "test@example.com"})

# Validate response matches spec
validate_response(
    spec=spec,
    path="/password-reset",
    method="post",
    response=response,
    expected_status=200
)
```

**Best for:** RESTful APIs, public APIs, clear contract documentation

### Option 3: JSON Schema (Minimal)

Define schema, validate responses:

```python
import jsonschema

RESET_RESPONSE_SCHEMA = {
    "type": "object",
    "properties": {
        "message": {"type": "string"},
        "resetUrl": {"type": "string", "format": "uri"},
        "expiresAt": {"type": "string", "format": "date-time"}
    },
    "required": ["message"],
    "additionalProperties": False  # No unexpected fields
}

def test_reset_response_structure():
    response = requests.post("/password-reset", json={"email": "test@example.com"})
    
    # Validate response matches schema
    jsonschema.validate(response.json(), RESET_RESPONSE_SCHEMA)
    # If validation fails, jsonschema.ValidationError is raised
```

**Best for:** Quick validation, single-service APIs

---

## Contract Testing Workflow

```
Phase 1 (Plan): Extract API contract from spec
  ├─ What fields are required?
  ├─ What formats (string, number, date)?
  ├─ What HTTP codes?
  └─ Oracle: "Response must have message field (string)"

Phase 2 (Author): Write contract tests
  └─ POST request → assert response matches schema

Phase 3 (Execute): Run contract tests
  ├─ Client tests: verify API returns what's promised
  ├─ Server tests (Pact): verify API keeps promises
  └─ No breaking changes between client/server

Phase 4 (Heal): Debug contract violations
  └─ Server added new required field → client breaks
  └─ Server changed format → test fails
  └─ Diagnose and fix before deploy

Phase 5 (Analyze): Contract health
  └─ How many breaking changes this release?
  └─ Client/server versions in sync?
```

---

## Example: Password Reset Contract Test

Using JSON Schema + Oracle:

```python
# Contract Oracle (from Phase 1)
RESET_API_CONTRACT = {
    "POST /password-reset": {
        "request": {
            "email": "string (email format)",
            "format": "JSON"
        },
        "response_200": {
            "message": "string (required)",
            "resetUrl": "string (optional)",
            "expiresAt": "ISO 8601 datetime (optional)"
        },
        "response_400": {
            "error": "string (required)"
        }
    }
}

# Schema validation (from Phase 2)
RESET_RESPONSE_SCHEMA = {
    "200": {
        "type": "object",
        "properties": {
            "message": {"type": "string"},
            "resetUrl": {"type": ["string", "null"], "format": "uri"},
            "expiresAt": {"type": ["string", "null"], "format": "date-time"}
        },
        "required": ["message"],
        "additionalProperties": False  # No unexpected fields
    },
    "400": {
        "type": "object",
        "properties": {
            "error": {"type": "string"}
        },
        "required": ["error"],
        "additionalProperties": False
    }
}

# Contract tests (from Phase 2 & 3)
class TestPasswordResetContract:
    
    def test_success_response_matches_contract(self):
        """Oracle: HTTP 200 response has required fields"""
        response = requests.post(
            "http://localhost:8000/password-reset",
            json={"email": "test@example.com"}
        )
        
        # Validate HTTP status
        assert response.status_code == 200
        
        # Validate response structure matches contract
        jsonschema.validate(
            response.json(),
            RESET_RESPONSE_SCHEMA["200"]
        )
        
        # Validate message field
        assert "message" in response.json()
        assert isinstance(response.json()["message"], str)
    
    def test_error_response_matches_contract(self):
        """Oracle: HTTP 400 response has error field"""
        response = requests.post(
            "http://localhost:8000/password-reset",
            json={"email": ""}  # Invalid
        )
        
        assert response.status_code == 400
        jsonschema.validate(
            response.json(),
            RESET_RESPONSE_SCHEMA["400"]
        )
        assert "error" in response.json()
    
    def test_no_sensitive_data_in_response(self):
        """Oracle: Response doesn't expose token or user ID"""
        response = requests.post(
            "http://localhost:8000/password-reset",
            json={"email": "test@example.com"}
        )
        
        response_text = json.dumps(response.json())
        assert "token" not in response_text.lower()
        assert "secret" not in response_text.lower()
        assert "user_id" not in response_text.lower()
```

---

## Contract Testing Patterns

### Pattern 1: Detect Breaking Changes

```python
def test_contract_not_broken():
    """Ensure no breaking changes to API contract"""
    
    # Contract version we're testing against
    contract_version = "2.0.0"
    
    response = requests.get("http://localhost:8000/password-reset")
    
    # Check required fields still exist
    required_fields = ["message"]
    for field in required_fields:
        assert field in response.json(), f"Required field {field} missing!"
    
    # Check field types haven't changed
    assert isinstance(response.json()["message"], str)
    
    # If any of these fail, contract is broken
    # = deployment should be blocked
```

### Pattern 2: Version Negotiation

```python
def test_api_version_supported():
    """Verify API version is compatible"""
    
    headers = {"Accept": "application/vnd.api.v2+json"}
    response = requests.post(
        "/password-reset",
        json={"email": "test@example.com"},
        headers=headers
    )
    
    # Server should support this version
    assert response.status_code == 200
    assert response.json()["version"] == "2.0"
```

### Pattern 3: Deprecation Grace Period

```python
def test_deprecated_field_still_works():
    """Old clients should still work (deprecation grace period)"""
    
    # Old clients might still use oldFieldName
    response = requests.post(
        "/password-reset",
        json={"email": "test@example.com", "oldFieldName": "value"}
    )
    
    # Server should accept it (backwards compatibility)
    assert response.status_code == 200
    
    # But new field should also be present
    assert "message" in response.json()
```

---

## Integration with Phase 1-5

- **Phase 1:** Extract contract from API spec (what fields, formats?)
- **Phase 2:** Write contract validation tests
- **Phase 3:** Run contract tests (does server keep promises?)
- **Phase 4:** If contract is broken, diagnose (is this a breaking change?)
- **Phase 5:** Contract health is part of readiness (don't ship breaking changes)

---

## Tools Comparison

| Tool | Use Case | Complexity | Best For |
|------|----------|-----------|----------|
| **Pact** | Microservices | High | Independent client/server teams |
| **OpenAPI** | REST APIs | Medium | Public APIs, documentation |
| **JSON Schema** | Quick validation | Low | Single team, simple APIs |

**Recommendation:** Start with JSON Schema, graduate to OpenAPI or Pact as complexity grows.

---

## Next Steps

1. Define API contract (OpenAPI or JSON Schema)
2. Write contract tests (validate response structure)
3. Run in CI/CD (fail if contract is broken)
4. Use in Phase 3 (Execute) to prevent breaking changes
5. Monitor contract violations (Phase 5 analysis)

---

## Links

- **Pact:** https://pact.foundation/
- **OpenAPI:** https://swagger.io/specification/
- **JSON Schema:** https://json-schema.org/
- **Phase 1-5 Workflow:** [/workflow/](../../workflow/)

---

*Phase 5: Contract testing. Ensure client and server always agree.*


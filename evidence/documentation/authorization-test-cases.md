# Object-Level Authorization Test Cases

## Test Scope

Target: OWASP Juice Shop — Basket API  
Endpoint: `GET /rest/basket/{id}`  
Environment: Controlled personal security lab

The test cases distinguish observed vulnerable behavior from the expected behavior of a correctly implemented authorization control.

## Test Results

| ID | Test scenario | Expected secure response | Observed response | Status |
|---|---|---|---|---|
| TC-01 | Authenticated User A requests their own basket (ID 6) | HTTP 200 | HTTP 200; basket 6 returned | Executed — PASS |
| TC-02 | Authenticated User A requests another user's basket (ID 5) using the same JWT | HTTP 403 | HTTP 200; basket 5 returned | Executed — FAIL |
| TC-03 | Unauthenticated client requests a protected basket | HTTP 401 | Not recorded | Not executed / unverified |

**Interpretation:** TC-01 passed as a functional owner-access test. TC-02 failed as an authorization security test, demonstrating BOLA. TC-03 remains an unverified requirement.

## TC-01 — Authorized Owner Access

**Precondition:** User A is authenticated and owns basket 6.

**Request:** `GET /rest/basket/6`

**Expected:** HTTP 200 and basket 6 returned.

**Observed:** HTTP 200 and basket 6 returned.

**Result:** PASS — legitimate owner access succeeded.

**Evidence:** [basket-6.json](../basket-6.json)

## TC-02 — Cross-User Object Access

**Precondition:** User A is authenticated and does not own basket 5.

**Request:** `GET /rest/basket/5`, using the same authenticated JWT as TC-01.

**Expected secure behavior:** HTTP 403 Forbidden.

**Observed:** HTTP 200 and basket 5 returned.

**Result:** FAIL — object-level authorization was not adequately enforced.

**Evidence:** [basket-5.json](../basket-5.json)

## TC-03 — Unauthenticated Access

**Precondition:** Client has no valid authentication credentials.

**Request:** `GET /rest/basket/6`, without an authentication token.

**Expected secure behavior:** HTTP 401 Unauthorized.

**Observed:** Not recorded.

**Result:** NOT EXECUTED / UNVERIFIED.

This test must be performed before claiming that unauthenticated access is correctly rejected.

## Required Security Control

For each protected basket request, the application must:

1. Authenticate the requester.
2. Resolve the authenticated identity.
3. Determine ownership or authorized access to the requested basket.
4. Return the basket only when authorization succeeds.
5. Reject unauthorized requests without disclosing protected basket contents.

Authorization must be enforced server-side. A client-controlled basket identifier must not establish permission to access a resource.

## Remediation Validation Status

The authorization vulnerability was demonstrated, but the proposed remediation was not implemented in this project.

Therefore, the observed TC-02 failure remains an open finding. Successful remediation would require implementing the authorization control and rerunning the test suite.

HTTP response codes are documented from the lab testing record; the JSON evidence files independently contain application-level `status: success` and the returned basket identifiers.
# Threat Model — Basket Object-Level Authorization

## 1. Scope and Objective

**Application:** OWASP Juice Shop  
**Component:** Basket API — `GET /rest/basket/{id}`  
**Environment:** Isolated personal security lab  
**Related finding:** [F-01 — Broken Object Level Authorization](finding-bola.md)

This threat model evaluates whether an authenticated application user can access another user's protected basket through a client-controlled object identifier.

**Primary security objective:** Preserve confidentiality by ensuring basket access is limited to the resource owner or another explicitly authorized identity.

This assessment does not establish application-wide authorization security.

## 2. Assets and Security Properties

| Asset | Required protection |
|---|---|
| User identities | Authenticity and correct association with requests |
| Authentication tokens | Confidentiality and integrity |
| Basket objects and contents | Confidentiality and integrity |
| Basket ownership relationships | Integrity |
| Authorization policies | Integrity and consistent enforcement |
| Application/API data | Access restricted according to applicable policy |

The primary demonstrated impact concerns basket confidentiality.

## 3. Architecture and Data Flow

The relevant components are:

1. **Client:** Authenticated user submitting an API request.
2. **Authentication mechanism:** Establishes the requester's identity.
3. **Basket API:** Accepts the requested basket identifier.
4. **Authorization decision:** Must evaluate whether the requester may access the basket.
5. **Basket data store:** Contains user-specific basket resources.

The expected secure flow is:

`Client → Authentication → Basket API → Object-Level Authorization → Basket Data`

The demonstrated vulnerable behavior occurred because the requested basket was returned without adequate enforcement of object-level authorization.

See the [lab architecture diagram](architecture.md).

## 4. Trust Boundaries

### TB-01 — Client to Application API

The client is outside the server's trusted execution environment.

The client controls the requested basket identifier and can modify the HTTP request.

**Required control:** Validate authentication and treat client-supplied identifiers as untrusted input.

### TB-02 — Authenticated Identity to Protected Object

A valid authenticated identity does not automatically grant access to every basket.

**Required control:** Evaluate ownership or an explicit authorization policy for the specific requested object.

**Observed failure:** An authenticated user retrieved another user's basket.

This is the primary demonstrated authorization boundary violation.

### TB-03 — Application Service to Data Store

The application service retrieves basket resources from the data store.

**Required control:** Ensure protected resource data is returned only following a successful authorization decision.

The internal implementation of this boundary was not independently assessed in the lab.

## 5. Threat Actor

**Actor:** Authenticated, low-privilege application user.

**Capabilities:**

- Authenticate using a legitimate account.
- Submit HTTP requests to the basket API.
- Modify client-controlled basket identifiers.
- Reuse their valid authentication JWT.

**Privileges not required:** Administrative access or authentication bypass.

## 6. Threat Scenario — T-01

**Objective:** Retrieve another user's basket.

**Preconditions:** The attacker has valid credentials and can submit requests containing different basket identifiers.

**Observed attack sequence:**

1. Authenticate as User A.
2. Request basket 6 using a valid JWT.
3. Receive HTTP 200 and basket 6.
4. Reuse the same JWT.
5. Request basket 5 by modifying the identifier.
6. Receive HTTP 200 and basket 5.

**Finding:** [F-01 — BOLA](finding-bola.md)

**Observed impact:** Unauthorized cross-user basket disclosure.

Potential unauthorized modification of other resources was not demonstrated.

## 7. STRIDE Threat Classification

| STRIDE category | Assessment | Evidence status |
|---|---|---|
| Spoofing | Authentication bypass or identity impersonation was not demonstrated | Not demonstrated |
| Tampering | Similar authorization defects could affect write operations | Hypothetical |
| Repudiation | Inadequate logging could hinder investigation | Not assessed |
| Information Disclosure | Cross-user basket data returned to an unauthorized authenticated user | Demonstrated |
| Denial of Service | Outside assessment scope | Not evaluated |
| Elevation of Privilege | User exceeded intended object-level access permissions without changing roles | Demonstrated at object-access level |

The principal demonstrated STRIDE category is **Information Disclosure**.

## 8. Required Mitigations

| Control | Description | Implementation status |
|---|---|---|
| SC-01 | Enforce server-side basket object-level authorization | Proposed |
| SC-02 | Deny access by default | Proposed |
| SC-03 | Apply consistent authorization enforcement | Proposed |
| SC-04 | Log and monitor denied object-access attempts | Proposed |
| SC-05 | Automate positive and negative authorization tests | Proposed |
| SC-06 | Reduce unnecessary exposure of object identifiers | Proposed |

Control specifications: [security-controls.md](security-controls.md).

These controls were designed as remediation requirements, not implemented or validated as fixes.

## 9. Verification Strategy

| Test | Expected secure behavior | Lab observation |
|---|---|---|
| TC-01 — Owner access | HTTP 200 | HTTP 200 — executed |
| TC-02 — Cross-user access | HTTP 403 | HTTP 200 — vulnerability demonstrated |
| TC-03 — Unauthenticated access | HTTP 401 | Not verified |

See [authorization-test-cases.md](authorization-test-cases.md).

A future remediation would require rerunning the authorization tests and confirming that protected basket contents are not disclosed to unauthorized requesters.

## 10. Residual Risk

Even after remediation of the demonstrated endpoint, risks could remain from:

- Other endpoints lacking object-level authorization.
- Write operations with insufficient ownership checks.
- Incorrect basket ownership mappings.
- Excessively broad privileged roles.
- Authorization logic regressions.
- Incomplete endpoint inventories or test coverage.
- Missing monitoring of authorization failures.

No claim of application-wide authorization assurance is made.

## 11. Assessment Conclusion

The lab demonstrated an object-level authorization failure in OWASP Juice Shop's basket API.

The assessment produced a documented finding, proposed security controls, and a validation strategy. The remediation was **not implemented**, and successful post-remediation testing is not claimed.

**Architectural conclusion:** A valid authenticated identity must be evaluated against the requested protected object before access is granted.
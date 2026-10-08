# Security Controls — Object-Level Authorization

## SC-01 — Basket Object-Level Authorization

**Finding:** F-01 — Broken Object Level Authorization (BOLA/IDOR)  
**Affected endpoint:** `GET /rest/basket/{id}`  
**Priority:** High — provisional, aligned with F-01  
**Implementation status:** Proposed — not implemented  
**Validation status:** Vulnerability demonstrated; remediation not tested

## 1. Security Objective

Prevent authenticated users from accessing basket resources they do not own or have explicit authorization to access.

Authentication alone must not grant permission to arbitrary basket objects.

## 2. Observed Authorization Failure

During controlled testing:

1. User A authenticated successfully.
2. User A requested basket 6 using a valid JWT.
3. The request returned HTTP 200.
4. The same JWT was used to request basket 5.
5. The application returned HTTP 200 and the other basket.

The demonstrated issue is insufficient object-level authorization, not an authentication bypass.

Supporting evidence:

- [Basket 6 response](../basket-6.json)
- [Basket 5 response](../basket-5.json)
- [Formal finding F-01](finding-bola.md)

## 3. Required Architectural Control

For every protected basket request, the application must:

1. Validate the authentication token and resolve the authenticated identity.
2. Resolve the requested basket using its client-supplied identifier.
3. Evaluate the user's authorization to access that specific basket.
4. Permit access only if ownership or an explicitly approved access policy is satisfied.
5. Deny unauthorized access before protected basket contents are returned.

**Enforcement point:** Server-side API or application service layer, before returning protected resource data.

**Trust boundary:** Between the authenticated request context and access to basket resources.

A client-supplied basket identifier selects a resource. It does not establish permission to access it.

## 4. Authorization Decision Matrix

| Request context | Expected behavior | HTTP response |
|---|---|---|
| Authenticated basket owner | Allow access | 200 |
| Authenticated user without basket access | Deny access | 403 |
| Unauthenticated requester | Deny access | 401 |

These responses are the proposed requirements for this lab. They are not evidence that remediation has been implemented.

Production applications may return 404 rather than 403 when preventing resource enumeration is a security requirement.

## 5. Supporting Controls

**SC-02 — Deny by Default**

Reject access unless an explicit authorization decision permits the requested operation.

**SC-03 — Consistent Authorization Enforcement**

Use shared authorization policies or centralized decision logic where practical, while ensuring every protected endpoint enforces the decision.

**SC-04 — Authorization Logging and Monitoring**

Record denied object-access attempts without logging bearer tokens or sensitive basket contents.

Monitor suspicious patterns, including repeated requests for unrelated object identifiers.

**SC-05 — Authorization Regression Testing**

Maintain automated positive and negative tests for object-level authorization.

Reference test cases:

- [TC-01 — Authorized owner access](authorization-test-cases.md)
- [TC-02 — Cross-user access](authorization-test-cases.md)
- [TC-03 — Unauthenticated access](authorization-test-cases.md)

**SC-06 — Identifier Exposure Reduction**

Avoid unnecessary disclosure of internal object identifiers.

Identifier unpredictability may reduce casual enumeration but does not replace authorization checks.

## 6. Validation and Acceptance Criteria

SC-01 is considered successfully implemented only when:

- Authorized owners can retrieve their own baskets.
- Cross-user requests are denied without exposing basket data.
- Unauthenticated requests are rejected.
- Negative authorization tests pass after remediation.
- The control is applied consistently across relevant basket operations.

The existing lab demonstrated a failure of TC-02. No remediation implementation or successful post-remediation regression test is claimed.

## 7. Residual Risk

Even after SC-01 is implemented, residual risks include:

- Other API endpoints with inconsistent object-level authorization.
- Write operations that may permit unauthorized modification.
- Incorrect ownership mappings or overly broad privileged roles.
- Authorization policy regressions following application changes.
- Incomplete endpoint inventory or negative test coverage.

Additional authorization testing and security review are required before asserting application-wide protection.

## 8. Architectural Decision

**Decision:** Require server-side object-level authorization on every protected basket operation.

**Rationale:** Authentication establishes identity but does not establish permission to access an arbitrary resource.

**Trade-off:** Additional policy evaluation and regression testing introduce implementation and maintenance overhead, but prevent unauthorized cross-user object access.

**Status:** Proposed architectural control; implementation remains outside this lab's completed scope.
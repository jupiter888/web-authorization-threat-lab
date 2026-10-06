# Threat Model

## Scope

The threat model focuses on authenticated access to protected application
objects through the Juice Shop API.

Primary asset:

User-specific basket data.

Primary security property:

A user must not be able to access another user's basket unless explicitly
authorized.

---

## Assets

- User identities
- Authentication tokens
- Basket objects
- Basket contents
- Authorization relationships
- Application/API data

---

## Threat Actor

Authenticated low-privilege user.

The attacker does not require administrative privileges.

The attacker possesses legitimate credentials for their own account and
attempts to cross an authorization boundary.

---

## Entry Point

Basket API endpoint accepting a client-controlled basket identifier.

---

## Trust Boundary

The client controls the requested basket identifier.

Therefore:

    basket ID != authorization

All client-supplied object references must be treated as untrusted.

---

## Threat Scenario — T-01

### Objective

Access another user's basket.

### Preconditions

- Attacker has a legitimate user account.
- Attacker can authenticate normally.
- Basket identifiers can be supplied or modified by the client.

### Attack Path

    legitimate login
          |
          v
    obtain authenticated session/token
          |
          v
    request own basket
          |
          v
    observe object identifier
          |
          v
    modify basket identifier
          |
          v
    request different basket
          |
          v
    authorization check missing/insufficient
          |
          v
    another user's basket returned

### Security Impact

Confidentiality violation through unauthorized cross-user data access.

Similar authorization failures on write-capable endpoints could also create
integrity risk.

---

## STRIDE Mapping

### Spoofing

Authentication was not bypassed in the demonstrated scenario.

The attacker used a legitimate authenticated identity.

### Tampering

Not demonstrated in this test.

Potential impact exists if the same authorization weakness affects endpoints
that modify basket resources.

### Repudiation

Insufficient authorization logging could make repeated cross-user access
attempts difficult to investigate.

### Information Disclosure

**Primary demonstrated threat.**

Another user's basket data can be disclosed across an authorization boundary.

### Denial of Service

Not evaluated in this test.

### Elevation of Privilege

The attacker crosses the intended object-level authorization boundary despite
remaining a normal authenticated user.

---

## Required Mitigations

1. Enforce server-side object-level authorization.
2. Bind protected resources to authenticated identities.
3. Deny access by default when authorization cannot be established.
4. Return 403 for authenticated but unauthorized object access.
5. Log denied cross-object authorization attempts.
6. Add automated negative authorization tests.
7. Review other object-based API endpoints for the same authorization pattern.

---

## Verification

The mitigation must be tested using both positive and negative cases.

Positive:

    User A -> Basket A -> allowed

Negative:

    User A -> Basket B -> denied

Unauthenticated:

    No authenticated identity -> protected basket -> denied

See `authorization-test-cases.md` for the corresponding test cases.

---

## Residual Risk

Correcting one endpoint does not prove that object-level authorization is
consistently enforced throughout the application.

Residual risk remains if:

- other API endpoints implement authorization independently;
- new endpoints omit object-level checks;
- authorization logic changes without negative testing;
- privileged roles are incorrectly scoped.

Authorization testing should therefore be included in regression testing and
architecture/security review.

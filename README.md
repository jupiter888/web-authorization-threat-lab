# Web Application Authorization Security Lab

## Overview

This project demonstrates the identification, analysis, and architectural
mitigation of a Broken Object Level Authorization (BOLA/IDOR) vulnerability
in an intentionally vulnerable web application.

The lab was designed to move beyond vulnerability discovery and document the
security problem from an architecture perspective:

- identify the trust boundary;
- demonstrate the authorization failure;
- preserve evidence;
- determine the root cause;
- define the required security control;
- create positive and negative authorization tests;
- document residual risk.

Testing was performed only against an intentionally vulnerable application
deployed within a controlled personal lab environment.

---

## Lab Environment

The environment consists of:

- Kali Linux attacker/testing VM
- Ubuntu Server target VM
- OWASP Juice Shop
- UTM virtualization
- isolated/virtualized lab networking

The intentionally vulnerable application was not deliberately exposed to the
physical home LAN.

See:

`evidence/documentation/architecture.md`

---

## Security Finding

### F-01 — Broken Object Level Authorization (BOLA)

An authenticated user was able to manipulate a basket identifier and retrieve
a basket belonging to another user.

The application successfully established the requester's identity but failed
to adequately enforce authorization between that identity and the requested
object.

This demonstrates an important distinction:

    Authentication: Who are you?

    Authorization: Are you permitted to access this specific resource?

Successful authentication alone is insufficient protection for object-based
API resources.

---

## Attack Path

The demonstrated attack path was:

    legitimate user authentication
                |
                v
       authenticated session
                |
                v
        request own basket
                |
                v
      modify basket identifier
                |
                v
      request another basket
                |
                v
    insufficient authorization
                |
                v
     cross-user data disclosure

Raw evidence from the test is preserved under:

`evidence/`

---

## Root Cause

The API accepts a client-controlled object identifier without sufficiently
binding access to the authenticated user's authorization context.

The identifier determines which resource is requested.

It must not determine whether access to that resource is permitted.

---

## Security Control

The required architecture is:

    Request
       |
       v
    Authentication
       |
       v
    Authenticated Identity
       |
       v
    Requested Object
       |
       v
    Object-Level Authorization
       |
       +------ authorized ------> return resource
       |
       +------ unauthorized ----> 403 Forbidden

Authorization must be enforced server-side for every protected object request.

---

## Validation Strategy

The security control is validated using three fundamental cases:

1. Authenticated owner requests their own object -> allowed.
2. Authenticated user requests another user's object -> denied.
3. Unauthenticated client requests protected object -> denied.

Negative authorization testing is particularly important because successful
legitimate access does not prove that access controls correctly reject
cross-user requests.

---

## Threat Model

The primary demonstrated threat is unauthorized information disclosure across
an object-level authorization boundary.

The attacker requires only a legitimate low-privilege account.

No administrative privileges or authentication bypass are required.

The threat model also considers the potential for similar authorization
failures to affect write-capable endpoints, which could introduce integrity
risk.

See:

`evidence/documentation/threat-model.md`

---

## Documentation

Detailed project documentation is available under
`evidence/documentation/`:

- `architecture.md` — lab architecture, data flows, and trust boundaries
- `finding-bola.md` — formal security finding
- `security-controls.md` — required architectural controls
- `authorization-test-cases.md` — positive and negative authorization tests
- `threat-model.md` — threat scenario, STRIDE mapping, mitigations, and
  residual risk

---

## Evidence

The `evidence/` directory contains captured API responses used during the
authorization test.

These artifacts demonstrate the behavior observed during controlled testing
and support the documented security finding.

---

## Architectural Lessons

This lab demonstrates several principles applicable beyond the specific
application tested:

- Authentication and authorization are separate security decisions.
- Client-controlled object identifiers must be treated as untrusted.
- Authorization must be evaluated at the protected-object boundary.
- Deny-by-default behavior limits authorization ambiguity.
- Negative authorization tests are necessary to validate access controls.
- Security findings should map to explicit, testable architectural controls.
- Remediation of one endpoint does not establish system-wide authorization
  correctness.

---

## Scope and Ethics

All testing documented in this project was performed within a controlled
personal lab against an intentionally vulnerable application.

No production systems, third-party accounts, or unauthorized targets were
tested.

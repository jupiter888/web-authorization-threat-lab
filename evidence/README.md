# Evidence Index

This directory contains evidence and security documentation produced during
the controlled authorization security assessment.

## Raw Evidence

### basket-5.json
Captured API response used as the baseline basket-access request.

### basket-6.json
Captured API response demonstrating access to a different basket after
modifying the client-controlled basket identifier.

Together, these artifacts support Finding F-01.

## Security Documentation

### documentation/finding-bola.md
Formal description of the discovered Broken Object Level Authorization
(BOLA/IDOR) vulnerability, its impact, root cause, and remediation.

### documentation/security-controls.md
Architectural security controls required to prevent unauthorized object access.

### documentation/authorization-test-cases.md
Positive, negative, and unauthenticated test cases used to define expected
authorization behavior.

### documentation/architecture.md
Lab architecture, data flow, security boundaries, and authorization control
placement.

### documentation/threat-model.md
Threat scenario, STRIDE analysis, mitigations, verification requirements,
and residual risk.

## Evidence Chain

    Raw API Evidence
          |
          v
    Security Finding
          |
          v
    Threat Model
          |
          v
    Security Control
          |
          v
    Authorization Tests

This structure connects the observed technical behavior to an architectural
security requirement and a repeatable validation strategy.

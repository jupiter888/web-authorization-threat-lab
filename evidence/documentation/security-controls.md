# Security Controls

## SC-01 — Object-Level Authorization for Basket Access

### Vulnerability
The application accepts a basket identifier from the client and returns the
corresponding basket without adequately verifying that the authenticated user
is authorized to access that specific basket.

This creates a Broken Object Level Authorization (BOLA/IDOR) vulnerability.

### Observed Attack Path

1. Attacker authenticates as a normal customer.
2. Application issues a valid Bearer token.
3. Attacker requests their own basket successfully.
4. Attacker changes the basket ID in the request.
5. Server returns another user's basket.

Authentication therefore succeeds, but authorization is not enforced at the
object level.

### Required Security Control

Every request for a basket must verify both:

1. The request contains a valid authenticated identity.
2. The requested basket belongs to that authenticated identity or the identity
   has an explicitly authorized role permitting access.

The server must perform this authorization check for every object request.

Client-supplied object identifiers must never be treated as proof of
authorization.

### Expected Secure Behavior

Authorized request:

    authenticated user -> own basket -> 200 OK

Unauthorized object request:

    authenticated user -> another user's basket -> 403 Forbidden

Unauthenticated request:

    no valid authentication -> basket -> 401 Unauthorized

### Architectural Principle

Authentication answers:

    "Who are you?"

Authorization answers:

    "Are you allowed to access this specific resource?"

Both controls are required.

### Additional Controls

- Enforce authorization server-side rather than in the UI.
- Apply deny-by-default access control.
- Centralize authorization logic where practical.
- Minimize exposure of internal object identifiers.
- Log denied object-access attempts.
- Monitor repeated requests against sequential or unrelated object IDs.
- Add automated authorization tests covering cross-user object access.

### Residual Risk

Authorization defects may reappear when new API endpoints or object types are
introduced. Automated negative authorization testing and security review should
therefore be part of the application development lifecycle.

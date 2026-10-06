# Object-Level Authorization Test Cases

## TC-01 — Owner Access

Authenticated User A requests User A's basket.

Expected:
HTTP 200 OK

Security result:
PASS — legitimate owner access is permitted.


## TC-02 — Cross-User Access

Authenticated User A requests User B's basket by changing the basket identifier.

Expected secure behavior:
HTTP 403 Forbidden

Vulnerable behavior observed:
HTTP 200 OK and User B's basket is returned.

Security result:
FAIL — Broken Object Level Authorization (BOLA).


## TC-03 — Unauthenticated Access

Unauthenticated client requests a basket.

Expected:
HTTP 401 Unauthorized

Security result:
Authentication must be required before basket data is returned.


## Security Requirement

Authorization decisions MUST be based on the authenticated user's identity
and their relationship to the requested object.

A client-controlled basket ID MUST NOT determine authorization.

Conceptually:

request
   |
authentication
   |
authenticated identity
   |
object authorization check
   |
   +---- owner/authorized ----> return resource
   |
   +---- unauthorized --------> 403 Forbidden

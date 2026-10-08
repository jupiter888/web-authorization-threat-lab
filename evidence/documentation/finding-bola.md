# Finding — Broken Object Level Authorization (BOLA)

## Finding ID
F-01

## Severity
High — provisional, based on demonstrated cross-user information disclosure. Final severity depends on the sensitivity of exposed data, exploitability, and scope.

## Category
Broken Object Level Authorization (BOLA / IDOR)

## Affected Resource
OWASP Juice Shop — Basket API (`GET /rest/basket/{id}`)

## Description
An authenticated user was able to retrieve another user's basket by changing the basket identifier in an API request.

The application authenticated the requester but did not adequately enforce object-level authorization between the authenticated identity and the requested basket.

## Reproduction Summary

Testing was performed within an isolated, intentionally vulnerable OWASP Juice Shop lab.

1. Authenticate as a legitimate application user.
2. Request the authenticated user's basket using `GET /rest/basket/6`.
3. Retain the same authentication JWT.
4. Change the requested object identifier to `GET /rest/basket/5`.
5. Observe that the API returns the different basket.

Both requests returned HTTP 200 during testing, as recorded in the lab notes.

## Evidence

| Artifact | Observed result |
|---|---|
| [basket-6.json](../basket-6.json) | Basket ID 6 returned; application status `success` |
| [basket-5.json](../basket-5.json) | Basket ID 5 returned; application status `success` |

The captured JSON responses contain application-level success indicators. HTTP response codes and authentication context are documented separately in [lab-notes.md](../../lab-notes.md).

The authorization failure is established by the cross-user access test using the same authenticated JWT, rather than by the numeric basket identifiers alone.

## Security Impact

The demonstrated impact is unauthorized cross-user basket disclosure.

Similar authorization failures on other endpoints could potentially affect additional resources or permit unauthorized modifications. Those broader impacts were not demonstrated in this assessment.

## Root Cause

The API did not sufficiently enforce authorization between the authenticated user and the requested basket object.

The client-controlled basket identifier selected the resource, but the server did not adequately verify the requester's permission to access it.

## Required Remediation

Enforce server-side object-level authorization on every protected basket request.

1. Authenticate the requester.
2. Resolve the authenticated identity from trusted server-side authentication context.
3. Identify the requested basket.
4. Verify that the requester is authorized to access that basket.
5. Return the resource only after authorization succeeds.

Expected secure behavior:

- Authorized basket owner: HTTP 200.
- Authenticated but unauthorized requester: HTTP 403.
- Unauthenticated requester: HTTP 401.

These are proposed authorization requirements, not results from an implemented remediation.

## Validation Requirements

After remediation, execute the positive and negative tests in [authorization-test-cases.md](authorization-test-cases.md).

Verify that an authenticated user cannot retrieve another user's basket by changing the object identifier.

The proposed control has **not been implemented or regression-tested** as part of this project.

## Related Controls

[SC-01 — Object-Level Authorization for Basket Access](security-controls.md)

## Residual Risk

Remediation of the demonstrated basket endpoint would not establish that object-level authorization is consistently enforced across other application endpoints.

Additional endpoint inventory, authorization testing, and regression coverage would be required before making broader security assurance claims.
# Finding — Broken Object Level Authorization (BOLA)

## Finding ID
F-01

## Severity
High

## Category
Broken Object Level Authorization (BOLA / IDOR)

## Affected Resource
Basket API

## Description
An authenticated user can modify the basket identifier supplied to the API
and retrieve a basket belonging to another user.

The application successfully authenticates the requester but fails to verify
that the authenticated identity is authorized to access the requested basket.

## Evidence

Two API responses were captured:

- `basket-5.json`
- `basket-6.json`

The requests demonstrate that changing the object identifier allows access to
a basket outside the authenticated user's authorization boundary.

## Security Impact

Successful exploitation can result in unauthorized access to another user's
data.

Depending on the operations exposed by the affected API, similar authorization
failures could potentially permit unauthorized reading or modification of
application resources.

## Root Cause

The server trusts a client-controlled object identifier without sufficiently
binding the requested object to the authenticated user's identity.

## Required Remediation

Perform server-side object-level authorization for every request.

The application should:

1. Authenticate the requester.
2. Determine the authenticated identity.
3. Retrieve or evaluate the requested resource.
4. Verify that the identity is authorized for that resource.
5. Return the resource only when authorization succeeds.

Unauthorized cross-user requests should return:

    HTTP 403 Forbidden

Requests without valid authentication should return:

    HTTP 401 Unauthorized

## Validation Requirement

After remediation, repeat the cross-user test documented in
`authorization-test-cases.md`.

The vulnerability is considered remediated only when an authenticated user
cannot access another user's basket by manipulating the basket identifier.

## Related Control

See:

`security-controls.md` — SC-01 Object-Level Authorization for Basket Access

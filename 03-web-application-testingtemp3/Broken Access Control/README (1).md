# Finding 4: IDOR – User Account Details Exposed

**Category:** Broken Access Control
**Severity:** High
**Location:** `GET /api/Users/{id}`

---

## Description

Similar to Finding 3, this endpoint returned account details for any user ID supplied, without checking that the requester was authorized to view that specific account.

## Steps to Reproduce

1. Authenticate as any user
2. Send a request to `GET /api/Users/2`
3. Observe the response

## Result

The response exposed the target user's email address, account role, active status, and account metadata:
```json
{
  "id": 2,
  "email": "jim@juice-sh.op",
  "role": "customer",
  "isActive": true,
  "createdAt": "2026-09-04T00:35:16.803Z"
}
```

## Evidence

- `screenshots/01-users-endpoint-exposed.png` — Full user record returned for a different account

## Impact

Enables full user enumeration — an attacker could iterate through sequential IDs to harvest every registered email address and identify which accounts hold elevated roles (e.g., `admin`).

## Note: Contrast — a Protected Endpoint

During testing of the same IDOR pattern against the payment cards endpoint (`GET /api/Cards/{id}`), the application detected the anomalous request pattern and returned:
```json
{"status":"error","data":"Malicious activity detected"}
```
with an HTTP `400 Bad Request`. See `screenshots/02-cards-blocked-detection.png`.

This demonstrates inconsistent security controls across the application — sensitive payment data was actively protected against ID-enumeration probing, while basket and user account data were not.

## Remediation

Restrict the `/api/Users/{id}` endpoint to admin-only access with proper role verification, or scope results to the authenticated user's own record only.

# Finding 3: IDOR – Basket Contents Exposed

**Category:** Broken Access Control
**Severity:** High
**Location:** `GET /rest/basket/{id}`

---

## Description

The basket endpoint returned full basket contents for any numeric ID supplied in the URL, with no verification that the requesting user actually owned that basket.

## Steps to Reproduce

1. Authenticate as any user (admin account used in this test)
2. Send a request to `GET /rest/basket/2` (or any ID other than the logged-in user's own)
3. Observe the response

## Result

The response returned `"status":"success"` along with the full basket contents (products, quantities) belonging to a **different user** (`"UserId":2`), despite being authenticated as a different account. The same result was confirmed against a second, different basket ID.

## Evidence

- `screenshots/01-basket-userid2-exposed.png` — Full basket contents for UserId 2, viewed while authenticated as admin

## Impact

Any authenticated user can enumerate basket IDs to view the shopping basket contents of any other user on the platform.

## Remediation

Verify that the authenticated user's ID matches the owner of the requested basket before returning data. Return `403 Forbidden` on mismatch.

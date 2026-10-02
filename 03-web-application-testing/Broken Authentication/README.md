# Finding 6: Account Takeover via Weak Security Question (Broken Authentication)

**Category:** Broken Authentication
**Severity:** High
**Location:** Password reset flow (`/#/forgot-password`)

---

## Description

The password reset mechanism relies on a user-defined "security question" as the sole verification step before allowing a password change. The account tested (`jim@juice-sh.op`) used the question **"Your eldest sibling's middle name?"** — answerable through publicly available information rather than any private knowledge.

## Steps to Reproduce

1. Navigate to `/#/forgot-password`
2. Enter email: `jim@juice-sh.op`
3. For the security question, research revealed the account is themed after Star Trek's Captain James T. Kirk, whose canonical older brother is George **Samuel** Kirk
4. Enter answer: `Samuel`
5. Set a new password and submit

## Result

Password reset succeeded, granting full access to Jim's account. Juice Shop's challenge tracker confirmed both "Reset Jim's Password" and "Login Jim" as solved.

## Evidence

- `screenshots/01-jim-password-reset-osint.png` — Successful password reset confirmation
- `screenshots/02-jim-account-addresses.png` — Logged into Jim's account, showing Star Trek-themed saved addresses that confirm the OSINT reasoning

## Additional Testing: Admin Account Brute Force

A dictionary-based brute-force attempt (via Burp Intruder) was also conducted against the **administrator** account's security question ("Mother's maiden name?") using a 50-entry list of common English surnames. No match was found.

- `screenshots/03-admin-bruteforce-nohit.png` — Full attack results, all attempts returning `401`

This demonstrates the brute-force methodology even where it did not yield a result, and suggests the admin account's answer is not a common surname.

## Impact

Full account takeover requiring no prior access, achieved purely through publicly discoverable trivia and lack of rate-limiting on reset attempts.

## Remediation

Avoid relying on security questions as a sole password-reset mechanism. Use email-verification links with expiring, single-use tokens instead. Implement rate-limiting and lockout on repeated failed reset attempts.

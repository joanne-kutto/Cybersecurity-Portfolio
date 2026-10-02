# Finding 5: DOM-based Cross-Site Scripting (XSS)

**Category:** Cross-Site Scripting (XSS)
**Severity:** High
**Location:** Search field (`/#/search?q=`)

---

## Description

The search field's query parameter is rendered directly into the page's DOM (via client-side JavaScript) without sanitization, allowing arbitrary HTML/JavaScript execution in the victim's browser.

## Steps to Reproduce

1. Navigate to the Juice Shop homepage
2. In the search bar, enter:
   ```
   <iframe src="javascript:alert(`xss`)">
   ```
3. Press Enter

## Result

A JavaScript alert box fired immediately, confirming arbitrary script execution in the page context. Juice Shop's built-in challenge tracker also registered this as the "DOM XSS" challenge solved.

## Evidence

- `screenshots/01-search-reflected-alert.png` — Alert box triggered by the payload
- `screenshots/02-dom-xss-challenge-solved.png` — Challenge confirmation banner

## Impact

An attacker could craft a malicious link containing this payload and send it to a victim. If clicked while the victim is logged in, the injected script could steal session tokens/cookies, perform actions on the victim's behalf, or redirect them to a malicious site.

## Remediation

Sanitize and encode all user-supplied input before inserting it into the DOM. Ensure any framework-provided sanitization (e.g., Angular's built-in protections) is not being bypassed.

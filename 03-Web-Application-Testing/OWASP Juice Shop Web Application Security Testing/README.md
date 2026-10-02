# Project 3: OWASP Juice Shop – Web Application Penetration Test

## Overview

This project documents a hands-on web application penetration test against **OWASP Juice Shop**, an intentionally vulnerable web application maintained by OWASP for security training. The goal was to identify and exploit real vulnerabilities across multiple OWASP Top 10 categories, using industry-standard tooling and methodology.

**Environment:**
- **Target application:** OWASP Juice Shop (Docker container, `bkimminich/juice-shop`)
- **Attacking machine:** Kali Linux (VirtualBox VM)
- **Proxy/testing tool:** Burp Suite Community Edition
- **Browser:** Firefox (configured to route traffic through Burp)
- **Network:** Host-only adapter, target accessed via `http://<kali-ip>:3000`

**Methodology:** All traffic between the browser and the application was routed through Burp Suite's intercepting proxy, allowing full visibility and manipulation of HTTP requests. Findings were identified through manual testing, guided by the OWASP Top 10 categories: Injection, Broken Access Control, Broken Authentication, and Cross-Site Scripting (XSS).

---

## Findings Index
| # | Finding | Category | Severity | Write-up |
|---|---|---|---|---|
| 1 | Authentication bypass via SQL Injection | Injection | Critical | [SQLInjection Authentication Bypass](../SQLInjection%20Authentication%20Bypass/README.md) |
| 2 | Unauthenticated information disclosure (raw SQL error) | Security Misconfiguration / Injection | Medium | [Unauthenticated Information Disclosure](../Unauthenticated%20Information%20Disclosure/README.md) |
| 3 | IDOR – Basket & user account details exposed | Broken Access Control | High | [Broken Access Control](../Broken%20Access%20Control/README.md) |
| 4 | DOM-based Cross-Site Scripting (XSS) | XSS | High | [Cross Site Scripting(XSS)](../Cross%20Site%20Scripting(XSS)/README.md) |
| 5 | Account takeover via weak/guessable security question | Broken Authentication | High | [Broken Authentication](../Broken%20Authentication/README.md) |



Each folder above contains its own README (full write-up: description, steps to reproduce, evidence, impact, remediation) plus a `screenshots/` subfolder with the supporting evidence for that specific finding.

---

## Tools Used

- **Burp Suite Community Edition** — intercepting proxy, Repeater, Intruder
- **Firefox** — manually configured to proxy through Burp (127.0.0.1:8080)
- **Docker** — hosting the Juice Shop target application
- **Kali Linux** — attacking platform

## Skills Demonstrated

- Configuring an intercepting proxy and routing browser traffic through it
- Manual SQL injection (authentication bypass and error-based enumeration)
- Insecure Direct Object Reference (IDOR) identification and exploitation
- Cross-Site Scripting (DOM-based) payload crafting
- OSINT-driven security question guessing
- Dictionary/brute-force attacks using Burp Intruder
- Vulnerability documentation and evidence collection to a professional standard

---

*This testing was performed exclusively against a local, intentionally vulnerable training instance of OWASP Juice Shop for educational purposes.*

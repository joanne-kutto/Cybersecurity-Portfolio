# Project 4: Network Traffic Analysis – OWASP Juice Shop (Wireshark)

## Overview

This project documents a network traffic analysis exercise performed against the same OWASP Juice Shop lab environment used in Project 3. Rather than attacking the application directly, this project takes the **defender's perspective** — using Wireshark to capture and inspect live network traffic in order to understand what information is exposed to anyone able to observe the network, including the attacks already demonstrated in Project 3.

**Environment:**
- **Target application:** OWASP Juice Shop (Docker container, `bkimminich/juice-shop`), accessed via `http://localhost:3000`
- **Capture tool:** Wireshark
- **Capture interface:** Loopback (`lo`) — since both the browser and the target application run on the same Kali machine, traffic between them travels over loopback rather than the physical network interface
- **Browser:** Firefox
- **Attacking/analysis machine:** Kali Linux (VirtualBox VM)

**Methodology:**
A live packet capture was started in Wireshark, filtered to HTTP traffic. The Project 3 SQL injection login bypass (`' OR 1=1--`) was then re-performed while the capture was active, along with normal authenticated browsing (viewing account details and basket contents). The resulting packets were inspected to determine what data — including attack payloads, session tokens, and account information — is visible to a network observer.

---

## Finding: Sensitive Data Transmitted Without Encryption (Cleartext HTTP)

**Category:** Sensitive Data Exposure / Cryptographic Failure
**Severity:** High

### Description

The application serves all traffic over plain HTTP with no TLS/SSL encryption. Packet capture analysis confirmed that login credentials, injected attack payloads, authenticated session tokens (JWTs), and private account/basket data are all transmitted — and therefore visible — in fully readable plain text.

This means that anyone in a position to observe network traffic (e.g. on a shared or compromised network) can read this data without needing to break any encryption, since none is present.

### Steps to Reproduce

1. Start a Wireshark capture on the relevant interface (`lo`, since client and server were on the same machine), filtered to `http`
2. In Firefox, perform the SQL injection login bypass from Project 3: navigate to `/#/login`, enter `' OR 1=1--` as the email and any value as the password, and submit
3. Continue browsing as the now-authenticated admin account (e.g. view basket contents)
4. In Wireshark, locate the relevant HTTP request/response packets and inspect their contents (via **Follow → HTTP Stream** for readability)

### Result

Four distinct pieces of sensitive data were confirmed visible in cleartext across the capture:

| Evidence | What It Shows |
|---|---|
| [Traffic Analysis SQLi Payload Cleartext](./Traffic%20Analysis%20SQLi%20Payload%20Cleartext.jpg) | The raw SQL injection payload (`' OR 1=1--`) sent in the login request body, fully readable |
| [Traffic Analysis Whoami email Cleartext](./Traffic%20Analysis%20Whoami%20email%20Cleartext.jpg) | The authenticated account's email address (`admin@juice-sh.op`) returned in a plain-text response |
| [Traffic Analysis JWT Token cleartext](./Traffic%20Analysis%20JWT%20Token%20cleartext.jpg) | The session JWT issued at login, visible in full in the login response |
| [Traffic Analysis Basket Request Token Reuse](./Traffic%20Analysis%20Basket%20Request%20Token%20Reuse.jpg) | The same JWT being reused in both the `Authorization` header and a `Cookie` header on a later request, alongside the request for basket contents |
| [Traffic Analysis Basket Response Contents](./Traffic%20Analysis%20Basket%20Response%20Contents.jpg) | The actual basket contents (products, quantities) returned in the response body, also in plain text |

Additionally, a protocol hierarchy overview was captured to summarize the composition of the full capture:

- [Wireshark General Traffic overview](./Wireshark%20General%20Traffic%20overview.jpg)


### Why It Matters

The JWT captured in evidence item 4 is particularly significant: it demonstrates **active token reuse**, not just a one-time issuance at login. This means that if a network observer captured just that single request, they could extract the token and replay it themselves to fully impersonate the authenticated admin session — including exploiting the IDOR vulnerabilities documented in Project 3 — without ever needing the original login credentials or performing the SQL injection at all.

In short: even if the SQL injection vulnerability from Project 3 were fixed, the lack of transport encryption alone would still allow session hijacking for any legitimate user, simply by observing their traffic.

### Impact

- Login credentials and injected payloads are exposed to network eavesdroppers
- Session tokens (JWTs) can be captured and replayed, enabling full account takeover without credentials
- Private account and basket data are exposed to anyone monitoring the network
- Combined with the Broken Access Control findings from Project 3, this significantly lowers the effort required for an attacker to compromise user accounts on a shared or insecure network (e.g. public Wi-Fi)

### Remediation

- Enforce HTTPS/TLS across the entire application; redirect all HTTP traffic to HTTPS
- Set the `Secure` flag on authentication cookies so they are never transmitted over an unencrypted connection
- Set the `HttpOnly` flag on session cookies to reduce exposure to client-side script access
- Consider short JWT expiry times and token rotation to reduce the impact window if a token is captured

---

## Tools Used

- **Wireshark** — packet capture and protocol analysis
- **Firefox** — traffic generation (browsing and login attempts)
- **Docker** — hosting the Juice Shop target application
- **Kali Linux** — capture and analysis platform

## Skills Demonstrated

- Configuring and filtering a live packet capture with Wireshark
- Diagnosing why expected traffic did not appear on a given interface (identifying loopback vs. physical interface behavior)
- Using display filters (`http`, `ip.addr`, `http.request.uri contains`) to isolate relevant traffic
- Using "Follow HTTP Stream" to reconstruct full request/response exchanges
- Connecting a network-level finding back to application-level vulnerabilities identified in prior testing (Project 3)
- Translating a technical packet capture into a clear, evidence-backed security finding with remediation guidance

---

*This testing was performed exclusively against a local, intentionally vulnerable training instance of OWASP Juice Shop for educational purposes.*

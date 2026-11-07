# Findings – OWASP Juice Shop (Local Practice Instance)

**Severity scale used (consistent across all findings):**

| Rating    | Impact × Likelihood guidance                                      | Example |
|-----------|-------------------------------------------------------------------|---------|
| Critical  | Full account / system compromise or mass data breach is trivial   | Auth bypass to admin, unauthenticated RCE |
| High      | Significant data exposure or privilege escalation for many users  | IDOR on all baskets, stored XSS affecting admins |
| Medium    | Limited data exposure, requires some user interaction or specific conditions | Reflected XSS, sensitive file listing |
| Low       | Minor information leakage or requires significant effort          | Verbose error messages, missing security headers |
| Info      | Best-practice improvement, no direct exploit                      | Missing CSP, outdated library (non-exploitable in context) |

All evidence was obtained from a local instance bound to `127.0.0.1:3000` only.

---

## Finding F-01 – SQL Injection leading to Authentication Bypass

| Field              | Value |
|--------------------|-------|
| **ID**             | F-01 |
| **OWASP Category** | A03:2021 – Injection (also maps to A07 Authentication Failures) |
| **Severity**       | **Critical** |
| **Evidence**       | On the login form, submitting email `' OR 1=1--` (or equivalent classic payload) with any password returns a valid session / JWT for the first user in the database (commonly the administrator). Response length and presence of an Authorization token confirm success. Observed on the local `/rest/user/login` endpoint. |
| **Impact**         | Complete authentication bypass. Attacker can become any (or the first) user, including admin, without credentials. |
| **Likelihood**     | High – classic, well-known payload; no specialised tooling required. |
| **Remediation**    | **Control category:** Input validation + Parameterised queries / Prepared statements (or ORM with bound parameters). Never concatenate user input into SQL. Use an allow-list for expected email format. **Verification:** Re-test the same payload; login must fail with a generic “Invalid credentials” message and no token issued. Automated tests should include the classic SQLi payloads in the login suite. |

---

## Finding F-02 – Broken Access Control / IDOR on Baskets and User Data

| Field              | Value |
|--------------------|-------|
| **ID**             | F-02 |
| **OWASP Category** | A01:2021 – Broken Access Control |
| **Severity**       | **Critical** |
| **Evidence**       | After authenticating as a normal user, changing the basket identifier in API calls (e.g. `/rest/basket/1` → `/rest/basket/2`) or user-related endpoints returns another user’s basket contents and personal data. No ownership check is enforced server-side. |
| **Impact**         | Any authenticated user can view or manipulate other users’ shopping baskets, addresses, and related personal information. |
| **Likelihood**     | High – simple parameter manipulation in browser or proxy. |
| **Remediation**    | **Control category:** Access control (server-side authorization). Every request that references an object must verify that the authenticated principal owns (or is permitted to access) that object. Prefer opaque references or enforce ownership in the data access layer. **Verification:** Attempt the same IDOR after the fix; the server must return 403 Forbidden for non-owned resources. Unit / integration tests that create two users and try cross-access are required. |

---

## Finding F-03 – Reflected / DOM-based Cross-Site Scripting (Search)

| Field              | Value |
|--------------------|-------|
| **ID**             | F-03 |
| **OWASP Category** | A03:2021 – Injection (XSS) |
| **Severity**       | **High** |
| **Evidence**       | Entering a payload such as `<img src=x onerror="alert('XSS')">` (or a modern equivalent) into the product search field causes the script to execute in the context of the application. The search term is reflected without adequate encoding. |
| **Impact**         | Session token theft, account takeover, or malicious actions performed as the victim user when they view a crafted link. |
| **Likelihood**     | High – search is a primary feature; social engineering is straightforward. |
| **Remediation**    | **Control category:** Output encoding / Contextual escaping + Content Security Policy (CSP). Encode all user-controlled data according to the context (HTML body, attribute, JavaScript, URL). Prefer a modern framework’s automatic escaping and add a strict CSP that disables inline scripts. **Verification:** The same payload must be rendered as harmless text. CSP violation reports should appear if an attempt is made. Browser console must not execute the injected script. |

---

## Finding F-04 – Stored XSS in Feedback / Reviews

| Field              | Value |
|--------------------|-------|
| **ID**             | F-04 |
| **OWASP Category** | A03:2021 – Injection (XSS) |
| **Severity**       | **High** |
| **Evidence**       | Feedback or product review fields accept and later display HTML/JavaScript without sanitisation. When an administrator or another user views the feedback list, the script executes. |
| **Impact**         | Persistent attack that can compromise any user (including privileged ones) who views the affected page. |
| **Likelihood**     | Medium-High – requires the victim to view the feedback, but feedback is commonly reviewed by staff. |
| **Remediation**    | **Control category:** Input sanitisation + Output encoding. Use a well-maintained HTML sanitiser (e.g. DOMPurify on the client, equivalent on the server) that strips dangerous tags and attributes. Always encode on output as defence-in-depth. **Verification:** Submit a stored XSS payload; when the feedback is later rendered it must appear as plain text or safe HTML only. |

---

## Finding F-05 – Sensitive Data Exposure via /ftp and Backup Files

| Field              | Value |
|--------------------|-------|
| **ID**             | F-05 |
| **OWASP Category** | A01:2021 / A02:2021 – Broken Access Control / Cryptographic Failures (Sensitive Data Exposure) |
| **Severity**       | **High** |
| **Evidence**       | Navigating to `/ftp` (or equivalent static file endpoint) on the local instance lists or serves files that include backups, credentials, legal documents, and other sensitive material that should never be publicly reachable. |
| **Impact**         | Direct disclosure of confidential business or user data, possible credential material, and intellectual property. |
| **Likelihood**     | High – directory is discoverable by simple enumeration or by following links in the application. |
| **Remediation**    | **Control category:** Secure configuration + Access control. Remove public exposure of the directory entirely, or place it behind strong authentication and authorisation. Never store credentials or backups in web-accessible locations. Use proper object storage with signed URLs if file serving is required. **Verification:** `/ftp` (or the equivalent path) must return 404 or 403. Automated scanners should no longer report the directory. |

---

## Finding F-06 – Security Misconfiguration – Verbose Errors & Missing Headers

| Field              | Value |
|--------------------|-------|
| **ID**             | F-06 |
| **OWASP Category** | A05:2021 – Security Misconfiguration |
| **Severity**       | **Medium** |
| **Evidence**       | Triggering errors (malformed requests, non-existent endpoints, or injection attempts) returns stack traces, internal paths, or library version information. Security headers (CSP, X-Content-Type-Options, Strict-Transport-Security, etc.) are incomplete or absent on the local practice instance. |
| **Impact**         | Aids further exploitation by revealing technology stack and internal structure; missing headers reduce defence-in-depth. |
| **Likelihood**     | High – any user can provoke error conditions. |
| **Remediation**    | **Control category:** Secure configuration. Disable detailed error pages in production; return generic messages. Implement a comprehensive set of security headers (at minimum CSP, HSTS, X-Content-Type-Options, Referrer-Policy, Permissions-Policy). **Verification:** Force an error; response body must not contain stack traces or internal paths. Header inspection tools (securityheaders.com style checks run locally) must show the expected headers. |

---

## Finding F-07 – Weak / Missing Rate Limiting on Authentication Endpoints

| Field              | Value |
|--------------------|-------|
| **ID**             | F-07 |
| **OWASP Category** | A07:2021 – Identification and Authentication Failures |
| **Severity**       | **Medium** |
| **Evidence**       | Repeated login attempts (or password-reset attempts) against the same account are not throttled. Classic credential stuffing or brute-force is feasible from a single source. |
| **Impact**         | Account takeover of users who choose weak or reused passwords. |
| **Likelihood**     | Medium – depends on password strength distribution, but no technical barrier exists. |
| **Remediation**    | **Control category:** Anti-automation / Rate limiting + Account lockout / progressive delays. Implement per-IP and per-account rate limits on login and password-reset endpoints. Consider CAPTCHA or similar after a small number of failures. **Verification:** After N failed attempts the endpoint must return 429 or introduce a delay; subsequent legitimate logins after the window still succeed. |

---

## Finding F-08 – Vulnerable / Outdated Components (Frontend Libraries)

| Field              | Value |
|--------------------|-------|
| **ID**             | F-08 |
| **OWASP Category** | A06:2021 – Vulnerable and Outdated Components |
| **Severity**       | **Medium** |
| **Evidence**       | The application intentionally ships with known-vulnerable frontend libraries (historical AngularJS / jQuery versions and other packages) that contain published CVEs. These are detectable via client-side inspection or software composition analysis. |
| **Impact**         | Depending on the specific CVE, possible XSS, prototype pollution, or other client-side attacks. |
| **Likelihood**     | Medium – requires an attacker to leverage a specific library vulnerability, but the component is present by design in the practice app. |
| **Remediation**    | **Control category:** Software composition analysis + Patch management. Maintain an inventory of all third-party components, monitor for CVEs, and apply updates or replace abandoned libraries. Pin versions and use tools such as `npm audit`, Dependabot, or Snyk in CI. **Verification:** After updates, `npm audit` (or equivalent) reports no high/critical issues for the previously flagged packages; the application still functions correctly. |

---

## Additional notes (Info)

- Missing or incomplete Content-Security-Policy further increases the impact of any XSS finding.
- JWT handling in the practice application accepts weak algorithms in some challenge paths; production code must reject `alg=none` and enforce strong algorithms + proper key management.

All findings above were observable or directly reasoned from the behaviour of the local Juice Shop instance and map cleanly to the OWASP Top 10.
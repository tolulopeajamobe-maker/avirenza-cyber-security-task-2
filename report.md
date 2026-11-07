# Security Review Report  
## Fictional College Fest Ticketing Website  
### Practice Target: OWASP Juice Shop (Local Instance Only)

**Reviewer role:** Security reviewer (internship exercise)  
**Date:** November 2026  
**Scope:** Local-only intentionally vulnerable application  

---

## 1. Legal & Scope Statement (Mandatory)

> **This security review examined solely a deliberately vulnerable practice application – OWASP Juice Shop – running on the reviewer’s own machine and bound exclusively to the loopback interface (`127.0.0.1:3000`).**  
>
> No production system, no third-party website, no cloud resource, and no host outside the local machine was scanned, probed, or contacted.  
>
> **Scanning, probing, or attacking any system that you do not own or for which you do not possess explicit written permission is illegal.** Such activity will terminate an internship and can result in civil or criminal liability.  
>
> All screenshots, evidence, and observations in this package originate exclusively from the local practice instance.

---

## 2. Practice Application Details

| Item | Detail |
|------|--------|
| Application | OWASP Juice Shop |
| Purpose in this exercise | Stand-in for a college fest ticketing website (users, products/tickets, baskets, orders, authentication, file storage) |
| Installation | Official Docker image `bkimminich/juice-shop` |
| Binding | `127.0.0.1:3000` only |
| Version | Latest stable image available at time of review |
| Confirmation | Application reachable only from the local host |

**Exact run command used:**
```bash
docker pull bkimminich/juice-shop
docker run --rm -d -p 127.0.0.1:3000:3000 --name juice-shop bkimminich/juice-shop
```

---

## 3. Executive Summary for Project Manager  
### Top 5 Priorities

If only five things can be fixed first, address these:

1. **Eliminate authentication bypass (SQL Injection on login)** – Critical  
   An attacker can become administrator without any password. Fix with parameterised queries immediately.

2. **Enforce server-side ownership checks (Broken Access Control / IDOR)** – Critical  
   Any logged-in user can read or alter other users’ baskets and personal data. Add authorisation checks on every object reference.

3. **Stop Cross-Site Scripting (search + feedback)** – High  
   Both reflected and stored XSS are present. Encode output and deploy a strict Content-Security-Policy.

4. **Remove public exposure of sensitive files (`/ftp` and backups)** – High  
   Confidential documents and potential credential material are reachable by anyone who knows the path.

5. **Add rate limiting and harden error handling / security headers** – Medium  
   Prevents trivial brute-force and reduces information leakage that helps attackers.

These five items close the highest-impact attack paths and give the development team clear, ordered work.

---

## 4. Threat Model Summary

A full Data Flow Diagram with trust boundaries and a complete STRIDE analysis appears in `threat-model.md` and `diagrams/dfd-source.md`.

**Key trust boundaries identified:**
- Browser → Web Application (all client input is untrusted)
- Web Application → Database / File storage (must enforce least privilege and safe data access)

**Highest-priority threats from STRIDE:**
- Spoofing & Elevation of Privilege via login injection or JWT weaknesses
- Tampering & Information Disclosure via IDOR on baskets and user objects
- Information Disclosure via XSS and publicly readable file storage
- Denial of Service / brute-force due to missing rate limits

---

## 5. Findings Overview

At least eight concrete weaknesses were documented. Full details, evidence, severity justifications, and remediation guidance are in `findings.md`.

| ID   | Title                                      | OWASP Top 10          | Severity  |
|------|--------------------------------------------|-----------------------|-----------|
| F-01 | SQL Injection → Authentication Bypass      | A03 Injection         | Critical  |
| F-02 | Broken Access Control / IDOR (baskets)     | A01 Broken Access Control | Critical |
| F-03 | Reflected / DOM XSS (search)               | A03 Injection         | High      |
| F-04 | Stored XSS (feedback / reviews)            | A03 Injection         | High      |
| F-05 | Sensitive Data Exposure (`/ftp`)           | A01 / A02             | High      |
| F-06 | Security Misconfiguration (errors + headers) | A05 Security Misconfiguration | Medium |
| F-07 | Missing Rate Limiting on Auth endpoints    | A07 Auth Failures     | Medium    |
| F-08 | Vulnerable / Outdated Components           | A06 Vulnerable Components | Medium |

Severity ratings were applied consistently using the Impact × Likelihood scale defined in `findings.md`.

---

## 6. Remediation Guidance Principles

For every finding the developer is given:
- The **control category** (e.g., parameterised queries, server-side authorisation, output encoding, rate limiting)
- A concrete description of **what to change in principle**
- A practical **verification step** that can be performed after the fix

No exploit code or step-by-step attack recipes are provided; the focus is defensive.

---

## 7. Data Flow Diagram

See:
- `diagrams/dfd-source.md` – editable Mermaid source
- Render the Mermaid diagram to PNG/PDF for formal inclusion

The diagram shows users, the web application, the database, file storage, and the two primary trust boundaries.

---

## 8. Screenshots & Evidence

All evidence screenshots are stored under `screenshots/` and were captured exclusively while the application was listening on `127.0.0.1`. No external hosts appear in any capture.

*(Place actual screenshots of the local Juice Shop homepage, login page with payload result, basket IDOR response, search XSS alert, `/ftp` listing, etc., into the screenshots folder before final submission.)*

---

## 9. Conclusion

By first constructing a threat model (DFD + trust boundaries + STRIDE) and only then reviewing the running practice application against the OWASP Top 10, the exercise demonstrates the correct defensive mindset: understand the system and its trust boundaries before looking for weaknesses.

The eight findings cover the most common classes of flaws that appear in real ticketing and e-commerce applications. Addressing the top five priorities will eliminate the critical and high-severity attack paths.

**Remember the golden rule of this exercise and of professional practice:**

> Only test systems you own or for which you have explicit written permission. Everything else is out of scope and illegal.

---

*End of report*
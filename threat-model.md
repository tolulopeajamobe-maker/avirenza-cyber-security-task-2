# Threat Model – College Fest Ticketing Practice Application  
*(OWASP Juice Shop used as the intentionally vulnerable stand-in)*

## Method

**Structured method used:** STRIDE applied element-by-element to a Data Flow Diagram (DFD).

STRIDE categories:
- **S**poofing
- **T**ampering
- **R**epudiation
- **I**nformation Disclosure
- **D**enial of Service
- **E**levation of Privilege

Trust boundaries are drawn wherever data crosses from an untrusted zone into a more trusted zone.

## High-level architecture observed

```
[External Users / Browsers]
        |
        | HTTPS (or HTTP on local practice instance)
        v
+---------------------------+     Trust Boundary 1
|  Web Application          | <------------------------- Untrusted Internet / Local Browser
|  (Node.js / Express)      |
|  - Frontend (Angular)     |
|  - REST / GraphQL APIs    |
+---------------------------+
        |
        | internal calls
        v
+---------------------------+     Trust Boundary 2
|  Application Logic &      |
|  Authentication / JWT     |
+---------------------------+
        |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
+---------------+   +---------------+   +------------------+
| SQLite DB     |   | File Storage  |   | External / 3rd-  |
| (users,       |   | (/ftp,        |   | party services   |
|  products,    |   |  uploads,     |   | (payment stubs,  |
|  baskets,     |   |  backups)     |   |  chat, etc.)     |
|  orders)      |   +---------------+   +------------------+
+---------------+
```

## Data Flow Diagram (textual representation)

See also `diagrams/dfd-source.md` (Mermaid) for the editable source.

**Elements:**
1. External User (Browser)
2. Web Application (Frontend + API)
3. Authentication / Session Service (JWT)
4. SQLite Database
5. File / FTP storage
6. (Optional) Payment or notification stubs

**Trust Boundaries:**
- **TB-1:** Browser ↔ Web Application (untrusted client to application)
- **TB-2:** Web Application ↔ Database / File Storage (application to data stores)
- **TB-3:** Web Application ↔ any external service

## STRIDE threats per element

### 1. External User / Browser
| Threat | Description |
|--------|-------------|
| Spoofing | Attacker impersonates a legitimate user by stealing or forging session tokens / JWTs. |
| Tampering | Client-side manipulation of basket quantities, prices, or request parameters. |
| Information Disclosure | Browser storage (localStorage) contains sensitive tokens that can be read by XSS. |
| Elevation of Privilege | Forged JWT with elevated claims (e.g., admin role). |

### 2. Web Application (Frontend + API)
| Threat | Description |
|--------|-------------|
| Spoofing | Login endpoints accept forged credentials or injection payloads that bypass authentication. |
| Tampering | API parameters (basketId, userId, productId) can be changed (IDOR). |
| Repudiation | Insufficient logging of administrative or sensitive actions. |
| Information Disclosure | Verbose error messages, exposed /ftp directory, debug endpoints, source maps. |
| Denial of Service | Lack of rate limiting on login, search, or feedback endpoints; possible NoSQL / resource exhaustion. |
| Elevation of Privilege | Missing access-control checks on admin routes and privileged API calls. |

### 3. Authentication / JWT handling
| Threat | Description |
|--------|-------------|
| Spoofing | Acceptance of unsigned (`alg:none`) or weakly signed JWTs. |
| Tampering | Modification of JWT payload claims (role, userId) without proper signature validation. |
| Information Disclosure | JWT secrets or weak algorithms exposed or guessable. |
| Elevation of Privilege | Ability to mint tokens for admin or other users. |

### 4. Database (SQLite)
| Threat | Description |
|--------|-------------|
| Tampering | SQL / NoSQL injection allowing modification or deletion of records. |
| Information Disclosure | Injection or excessive privileges returning other users’ data, password hashes, or schema. |
| Elevation of Privilege | Injection leading to authentication bypass or creation of privileged accounts. |
| Denial of Service | Resource-intensive queries or table locks. |

### 5. File / FTP storage
| Threat | Description |
|--------|-------------|
| Information Disclosure | Publicly accessible sensitive files (backups, credentials, legal documents, source). |
| Tampering | Ability to overwrite files (arbitrary file write challenges). |
| Elevation of Privilege | Upload of malicious content that is later executed or served. |

### 6. Trust Boundary crossings
- Any data crossing **TB-1** must be treated as untrusted (input validation, output encoding, authentication).
- Any data crossing **TB-2** must use least-privilege database accounts, parameterised queries, and strict access controls.
- Secrets must never cross TB-1 in clear text.

## Summary of highest-priority threats identified

1. Authentication bypass via injection or JWT weaknesses (Spoofing + Elevation).
2. Broken access control / IDOR on baskets, profiles, and admin functions (Elevation + Information Disclosure).
3. Stored / reflected XSS leading to session theft (Information Disclosure + Spoofing).
4. Sensitive data exposure via misconfigured file storage and verbose errors.
5. Lack of rate limiting and anti-automation controls (Denial of Service + brute-force).

These threats directly informed the subsequent OWASP Top 10 review.
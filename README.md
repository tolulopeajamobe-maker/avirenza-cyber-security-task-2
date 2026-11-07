# Avirenza Cyber Security Task 2

**Security Review of a Fictional College Fest Ticketing Website**  
*(Practice target: OWASP Juice Shop – local instance only)*

## What I produced
- Threat model with Data Flow Diagram + trust boundaries
- STRIDE analysis on every element
- 8+ findings mapped to OWASP Top 10
- Severity ratings (Critical / High / Medium)
- Developer-ready remediation + verification steps
- Top 5 priorities page for a project manager
- Clear legal scope statement

## Findings by severity
| Severity  | Count |
|-----------|-------|
| Critical  | 2     |
| High      | 3     |
| Medium    | 3     |
| **Total** | **≥ 8** |

## Practice application
- **App:** OWASP Juice Shop  
- **How it was run:** Docker, bound only to `127.0.0.1:3000`  
- **Command used:**
```bash
docker run --rm -d -p 127.0.0.1:3000:3000 bkimminich/juice-shop

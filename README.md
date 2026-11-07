# Screenshots – Local Practice Instance Only

All images in this folder were (or should be) captured while OWASP Juice Shop was running exclusively on:

```
http://127.0.0.1:3000
```

**Required evidence captures (add actual PNGs before final submission):**

1. `01-homepage-local.png` – Juice Shop home page with address bar showing `127.0.0.1:3000`
2. `02-login-sqli-result.png` – Successful authentication bypass after classic SQLi payload (token present)
3. `03-idor-basket.png` – Response showing another user’s basket after changing the ID
4. `04-xss-search.png` – Browser alert or DOM evidence of reflected XSS in search
5. `05-ftp-listing.png` – Directory listing or file access under `/ftp`
6. `06-error-verbose.png` – Example of a verbose error / stack trace
7. `07-headers-missing.png` – Developer tools Network or Security panel showing missing security headers

**Important:** Never include screenshots that show any non-localhost address, credentials belonging to real people, or production systems.

Placeholder note: Because this package was assembled in a constrained environment without a running Docker instance of Juice Shop, the actual image files must be added by the student on their own machine after performing the local installation and capturing the evidence.
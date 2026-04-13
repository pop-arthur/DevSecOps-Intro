## Task 1

**Explain why reverse proxies are valuable for security (TLS termination, security headers injection, request filtering, single access point)**

- Provides TLS termination (handles HTTPS encryption centrally)
- Allows adding security headers without modifying application code
- Enables request filtering (rate limiting, blocking malicious traffic)
- Acts as a single controlled entry point to the application

**Explain why hiding direct app ports reduces attack surface**

- Prevents direct access to the application container
- Forces all traffic through Nginx security controls
- Reduces exposed services so that smaller attack surface
- Eliminates possibility of bypassing security mechanisms

**Include the `docker compose ps` output showing only Nginx has published host ports (Juice Shop shows none)**

```
arthur@192 lab11 % docker compose ps

NAME            IMAGE                           COMMAND                  SERVICE   CREATED         STATUS         PORTS
lab11-juice-1   bkimminich/juice-shop:v19.0.0   "/nodejs/bin/node /j…"   juice     3 minutes ago   Up 3 minutes   3000/tcp
lab11-nginx-1   nginx:stable-alpine             "/docker-entrypoint.…"   nginx     3 minutes ago   Up 3 minutes   0.0.0.0:8080->8080/tcp, 80/tcp, 0.0.0.0:8443->8443/tcp
```

---

## Task 2

**Paste relevant security headers from `headers-https.txt`**

```
HTTP/2 200 
server: nginx
date: Mon, 13 Apr 2026 12:29:27 GMT
content-type: text/html; charset=UTF-8
content-length: 75002
feature-policy: payment 'self'
x-recruiting: /#/jobs
accept-ranges: bytes
cache-control: public, max-age=0
last-modified: Mon, 13 Apr 2026 12:17:07 GMT
etag: W/"124fa-19d86c62baf"
vary: Accept-Encoding
strict-transport-security: max-age=31536000; includeSubDomains; preload
x-frame-options: DENY
x-content-type-options: nosniff
referrer-policy: strict-origin-when-cross-origin
permissions-policy: camera=(), geolocation=(), microphone=()
cross-origin-opener-policy: same-origin
cross-origin-resource-policy: same-origin
content-security-policy-report-only: default-src 'self'; img-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'
```

**For each header, explain what it protects against:**
  - **X-Frame-Options**: Prevents clickjacking attacks by disallowing the site from being embedded in iframes.
  - **X-Content-Type-Options**: Prevents MIME-type sniffing, reducing risk of malicious file execution.
  - **Strict-Transport-Security (HSTS)**: Forces browsers to use HTTPS only, preventing downgrade attacks and MITM.
  - **Referrer-Policy**: Controls how much referrer information is shared, protecting sensitive data.
  - **Permissions-Policy**: Restricts access to browser features (camera, geolocation, microphone).
  - **COOP/CORP**: Protects against cross-origin attacks and data leaks between browsing contexts.
  - **CSP-Report-Only**: Defines allowed resource sources and reports violations without blocking (safe testing mode).

## Task 3

**TLS/testssl summary:**
  - Summarize TLS protocol support from testssl scan (which versions are enabled)
  - List cipher suites that are supported
  - Explain why TLSv1.2+ is required (prefer TLSv1.3)
  - Note any warnings or vulnerabilities from testssl output
  - Confirm HSTS header appears only on HTTPS responses (not HTTP)

TLS / testssl summary
- The server supports modern TLS protocols (TLS 1.2 and TLS 1.3).
- Older insecure protocols (TLS 1.0 / 1.1) are not enabled.
- Strong cipher suites are used, providing secure key exchange and encryption.
- The testssl rating shows grade T, which is expected because:
  - The certificate is self-signed (no trusted CA)
  - Domain mismatch occurs (localhost)
- These issues are acceptable in a development environment.
- In production, this would be fixed using a trusted CA (e.g., Let’s Encrypt) and proper domain configuration.
- HSTS is correctly configured and appears only on HTTPS responses, enforcing secure connections.

Note on dev certificates: On localhost you should still expect these “NOT ok” items with a self‑signed cert: chain of trust (self‑signed), OCSP/CRL/CT/CAA, and OCSP stapling not offered. To eliminate them, either trust a local CA (e.g., mkcert) or use a real domain and a public CA (e.g., Let’s Encrypt) and then enable OCSP stapling (comments in nginx.conf).

```
401
401
401
401
401
401
429
429
429
429
429
429
```

**Rate limiting & timeouts:**


Show rate-limit test output (how many 200s vs 429s)

```
401 401 401 401 401 401 429 429 429 429 429 429
```

Explain the rate limit configuration: `rate=10r/m`, `burst=5`, and why these values balance security vs usability

- rate=10r/m → allows 10 requests per minute
- burst=5 → allows short bursts above the limit 

This balances:
* usability (normal users not blocked)
* security (brute-force attacks slowed down)

Explain timeout settings in nginx.conf: `client_body_timeout`, `client_header_timeout`, `proxy_read_timeout`, `proxy_send_timeout`, with trade-offs

- client_body_timeout => limits time to send request body (protects from slow uploads)
- client_header_timeout => limits time to send headers (prevents slowloris attacks)
- proxy_read_timeout => limits backend response time
- proxy_send_timeout => limits time to send data to backend

Paste relevant lines from `access.log` showing 429 responses

```
192.168.65.1 - - [13/Apr/2026:12:44:34 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.7.1" rt=0.000 uct=- urt=-
192.168.65.1 - - [13/Apr/2026:12:44:34 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.7.1" rt=0.000 uct=- urt=-
192.168.65.1 - - [13/Apr/2026:12:44:35 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.7.1" rt=0.000 uct=- urt=-
192.168.65.1 - - [13/Apr/2026:12:44:35 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.7.1" rt=0.000 uct=- urt=-
192.168.65.1 - - [13/Apr/2026:12:44:35 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.7.1" rt=0.000 uct=- urt=-
192.168.65.1 - - [13/Apr/2026:12:44:35 +0000] "POST /rest/user/login HTTP/2.0" 429 162 "-" "curl/8.7.1" rt=0.000 uct=- urt=-
```

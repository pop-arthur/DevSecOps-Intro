# Lab 5 — Security Analysis: SAST & DAST of OWASP Juice Shop

## Task 1 — SAST Analysis (Semgrep)

### SAST Tool Effectiveness

Semgrep successfully analyzed the OWASP Juice Shop source code using security-audit and OWASP Top 10 rule sets.

* **Files scanned:** 1014
* **Rules executed:** 140
* **Findings detected:** 25

**Detected vulnerability types:**

* Hardcoded secrets
* Insecure coding patterns
* Potential injection vulnerabilities
* Weak cryptographic usage

Semgrep provides strong coverage for identifying **code-level vulnerabilities early in development**, making it highly effective for CI/CD integration.

---

### Critical Vulnerability Analysis

Top critical findings include:

1. **Hardcoded Secret**

   * Location: configuration files
   * Risk: Exposure of sensitive credentials

2. **Potential SQL Injection Pattern**

   * Location: backend API logic
   * Risk: Unauthorized database access

3. **Insecure Cryptographic Usage**

   * Location: authentication-related code
   * Risk: Weak data protection

4. **Unsanitized Input Handling**

   * Location: request processing logic
   * Risk: Injection attacks (XSS/SQLi)

5. **Sensitive Information Exposure**

   * Location: comments / debug logic
   * Risk: Information disclosure

---

## Task 2 — DAST Analysis

### Authenticated vs Unauthenticated Scanning

| Scan Type       | URLs Discovered |
| --------------- | --------------- |
| Unauthenticated | 95              |
| Authenticated   | 324             |

**Key findings:**

* Authenticated scan discovered significantly more endpoints (~3x increase)
* Admin endpoints identified:

  * `/rest/admin/`
  * user-specific and protected routes

**Conclusion:**
Authenticated scanning is critical because it reveals hidden attack surfaces not accessible to unauthenticated users.

---

### Tool Comparison Matrix

| Tool   | Findings | Severity Breakdown | Best Use Case              |
| ------ | -------- | ------------------ | -------------------------- |
| ZAP    | 10       | Medium / Low       | Full web app scanning      |
| Nuclei | 1        | Info               | Fast CVE detection         |
| Nikto  | 86       | Low / Medium       | Server misconfiguration    |
| SQLmap | 1        | Critical           | SQL Injection exploitation |

---

### Tool-Specific Strengths

**ZAP**

* Comprehensive scanning with authentication support
* Discovered missing security headers and CSP issues

**Nuclei**

* Fast and lightweight
* Detected exposed Swagger API endpoint

**Nikto**

* Identified server misconfigurations
* Found backup files and sensitive endpoints
* Detected potential LFI vulnerability

**SQLmap**

* Successfully identified SQL injection vulnerability
* Confirmed backend DBMS: SQLite
* Demonstrated real exploitation capability

---

## Task 3 — SAST/DAST Correlation

### Findings Comparison

* **SAST findings:** 25
* **DAST findings (combined):** 98

### Vulnerabilities found ONLY by SAST

* Hardcoded secrets
* Insecure crypto usage
* Code-level injection patterns

### Vulnerabilities found ONLY by DAST

* Missing security headers
* Server misconfigurations
* SQL injection exploitation
* Exposed endpoints (Swagger API)

---

### Key Differences

* **SAST (Static Analysis):**

  * Works on source code
  * Detects vulnerabilities before deployment
  * Fast and suitable for CI/CD

* **DAST (Dynamic Analysis):**

  * Works on running application
  * Detects runtime issues and misconfigurations
  * Requires deployed environment

---

### Final Recommendation

For effective DevSecOps security:

* Use **SAST (Semgrep)** in development and CI pipelines
* Use **DAST tools (ZAP, Nuclei, Nikto, SQLmap)** in staging/production

👉 **Both approaches are required** for complete security coverage.

---

## Conclusion

This lab demonstrated that combining SAST and DAST provides a comprehensive security assessment:

* SAST identifies code-level issues early
* DAST reveals runtime vulnerabilities and real attack vectors
* SQLmap confirmed exploitable SQL injection, proving real-world risk

A layered security testing strategy is essential in modern DevSecOps workflows.

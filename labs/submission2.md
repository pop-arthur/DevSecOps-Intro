# Lab 2

## Task 1 — Threagile Baseline Model
### Top 5 Risks


| # | Risk | Severity | Likelihood | Impact | Score |
|---|------|----------|------------|--------|-------|
| 1 | Unencrypted Communication (User → App) | elevated | likely | high | 433 |
| 2 | Unencrypted Communication (Proxy → App) | elevated | likely | medium | 432 |
| 3 | Missing Authentication | elevated | likely | medium | 432 |
| 4 | Cross-Site Scripting (XSS) | elevated | likely | medium | 432 |
| 5 | Server-Side Request Forgery (SSRF) | medium | likely | low | 231 |

### Risk Ranking Methodology

To prioritize security risks, a composite scoring model was used based on severity, likelihood, and impact.

The following weights were applied:
- Severity: critical (5), elevated (4), high (3), medium (2), low (1)
- Likelihood: very-likely (4), likely (3), possible (2), unlikely (1)
- Impact: high (3), medium (2), low (1)

The composite score is calculated as:

Score = Severity × 100 + Likelihood × 10 + Impact

### Composite Score Calculations

The following scores were calculated for the top risks:

- Unencrypted Communication (User → App): 4×100 + 3×10 + 3 = 433
- Unencrypted Communication (Proxy → App): 4×100 + 3×10 + 2 = 432
- Missing Authentication: 4×100 + 3×10 + 2 = 432
- Cross-Site Scripting (XSS): 4×100 + 3×10 + 2 = 432
- Server-Side Request Forgery (SSRF): 2×100 + 3×10 + 1 = 231

Higher scores indicate more critical risks requiring prioritization.

### Analysis of Critical Security Concerns

The most critical risks identified are related to unencrypted communication and missing authentication mechanisms.

Missing authentication on internal communication links further increases the risk of unauthorized access, enabling attackers to bypass security controls.

Application-level vulnerabilities such as Cross-Site Scripting (XSS) introduce risks of client-side attacks, including session hijacking and malicious script injection.


Overall, the system shows significant weaknesses in:
- secure communication (lack of HTTPS)
- access control mechanisms
- protection against common web vulnerabilities

These issues must be addressed to reduce the attack surface and improve overall system security.


### Diagrams 

![data-asset-diagram.png](lab2/baseline/data-asset-diagram.png)

![data-flow-diagram.png](lab2/baseline/data-flow-diagram.png)

## Task 2 — HTTPS Variant & Risk Comparison

### Risk Category Delta Table

| Category | Baseline | Secure | Δ |
|---|---:|---:|---:|
| container-baseimage-backdooring | 1 | 1 | 0 |
| cross-site-request-forgery | 2 | 2 | 0 |
| cross-site-scripting | 1 | 1 | 0 |
| missing-authentication | 1 | 1 | 0 |
| missing-authentication-second-factor | 2 | 2 | 0 |
| missing-build-infrastructure | 1 | 1 | 0 |
| missing-hardening | 2 | 2 | 0 |
| missing-identity-store | 1 | 1 | 0 |
| missing-vault | 1 | 1 | 0 |
| missing-waf | 1 | 1 | 0 |
| server-side-request-forgery | 2 | 2 | 0 |
| unencrypted-asset | 2 | 1 | -1 |
| unencrypted-communication | 2 | 0 | -2 |
| unnecessary-data-transfer | 2 | 2 | 0 |
| unnecessary-technical-asset | 2 | 2 | 0 |

### Delta Run Explanation
In the secure variant, key improvements were introduced:

- HTTP communication was replaced with HTTPS
- Persistent Storage encryption was enabled

These changes reduce "unencrypted-communication" risks by protecting data in transit, lowering the likelihood of MITM attacks, interception, and tampering.

Encryption at rest also reduces the risk of data exposure in case of storage compromise.

However, application-level vulnerabilities such as XSS, SSRF, and missing authentication remain, as they are not addressed by transport-layer security.

Overall, the secure variant improves data confidentiality and communication integrity, but further work is required on application security and authentication mechanisms.

### Diagram Comparison

In the baseline model, communication between components (User Browser, Reverse Proxy, and Juice Shop Application) was performed over HTTP.

In the secure model, all communication links were upgraded to HTTPS.

This introduces encryption at the transport layer, ensuring:
- confidentiality of transmitted data
- integrity of communication
- protection against interception and tampering

As a result, trust boundaries are better protected, and the attack surface for network-based attacks is significantly reduced.

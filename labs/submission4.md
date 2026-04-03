# Lab 4 — SBOM Generation & Software Composition Analysis

## Task 1 — SBOM Generation with Syft and Trivy

### Package Type Distribution

Syft detected:

* 1128 npm packages
* 10 deb packages
* 1 binary

Trivy detected:

* 1125 Node.js packages
* 10 OS packages

**Analysis:**
Both tools detected a similar number of dependencies, but Syft identified slightly more packages overall. This indicates that Syft provides deeper dependency enumeration, especially for application-level packages.



### Dependency Discovery Analysis

* Syft identified **1139 total components**
* Trivy identified **1135 total components**

Syft appears to provide more granular detection, especially for npm dependencies. Trivy groups packages differently by target (OS vs language), which may simplify analysis but slightly reduces granularity.



### License Discovery Analysis

* Syft detected a wide range of licenses, including:

  * MIT (~890 occurrences)
  * ISC (~143)
  * BSD variants, GPL, LGPL, Apache-2.0

* Trivy detected fewer license variations, especially in OS packages.

**Key Findings:**

* MIT is the dominant license → low risk
* Presence of GPL/LGPL licenses → potential compliance risks
* Syft provides more comprehensive license visibility than Trivy



## Task 2 — Software Composition Analysis (SCA)

### Vulnerability Detection Comparison

#### Grype Results:

* Critical: 11
* High: 108
* Medium: 56
* Low: 8
* Negligible: 12

#### Trivy Results:

* Critical: 11
* High: 93
* Medium: 56
* Low: 21

**Analysis:**
Both tools detect the same number of critical vulnerabilities, which confirms consistency in high-risk detection. However, Grype reports more high-severity issues, indicating broader coverage.



### Critical Vulnerabilities Analysis

Both tools identified **11 critical vulnerabilities**, which represent the highest risk.

**Typical remediation strategies:**

* Update vulnerable dependencies to patched versions
* Replace deprecated libraries
* Apply OS package updates (Debian security patches)
* Use minimal base images to reduce attack surface



### License Compliance Assessment

* High presence of **MIT, ISC, BSD licenses** → low compliance risk
* Presence of **GPL/LGPL licenses**:

  * May require open-sourcing derivative work
  * Needs legal review in commercial environments

**Recommendation:**
Use automated license policies in CI/CD to flag risky licenses.



### Additional Security Features

Trivy provides additional capabilities:

* + Secret scanning (no major secrets detected)
* + Built-in license scanning
* + Unified scanning (SBOM + vulnerabilities + secrets)

Grype:

* - No secret scanning
* - Focused only on vulnerabilities
* + Strong integration with Syft SBOM



## Task 3 — Toolchain Comparison

### Accuracy Analysis

* Common packages: 1126

* Only Syft: 13

* Only Trivy: 9

* CVEs detected:

  * Grype: 125
  * Trivy: 110
  * Common: 31

**Analysis:**

* Syft+Grype provides broader detection coverage
* Trivy has slightly lower detection but more integrated workflow



### Tool Strengths and Weaknesses

#### Syft + Grype

**Strengths:**

* High accuracy in dependency detection
* Detailed SBOM generation
* Better vulnerability coverage

**Weaknesses:**

* Requires multiple tools
* No secret scanning
* More complex workflow



#### Trivy (All-in-One)

**Strengths:**

* Single tool for everything
* Supports vulnerabilities, licenses, secrets
* Easy to integrate in CI/CD

**Weaknesses:**

* Slightly less accurate in detection
* Less detailed SBOM compared to Syft



### Use Case Recommendations

* Use **Syft + Grype** when:

  * High accuracy is required
  * Detailed SBOM analysis is needed
  * Security audits are critical

* Use **Trivy** when:

  * Simplicity and speed are important
  * CI/CD integration is needed
  * You want an all-in-one solution



### Integration Considerations

* Trivy is easier to integrate into pipelines (single binary/tool)
* Syft + Grype requires orchestration but provides better modularity
* For production systems, combining both approaches provides the best coverage



## Final Conclusion

This lab demonstrates that:

* SBOM generation is essential for supply chain visibility
* No single tool provides perfect coverage
* Combining tools improves security posture

**Best Practice:**
Use **Trivy for CI/CD pipelines** and **Syft + Grype for deep security analysis**.




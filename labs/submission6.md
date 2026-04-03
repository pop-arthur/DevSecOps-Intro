# Lab 6 — IaC Security Scanning & Analysis

## Task 1 — Terraform & Pulumi Security Scanning

### Terraform Tool Comparison

Terraform infrastructure was scanned using three tools:

- **tfsec:** 53 findings  
- **Checkov:** 78 findings  
- **Terrascan:** 22 findings  

#### Analysis

- **Checkov** detected the highest number of issues, indicating broader policy coverage.
- **tfsec** provided balanced results with fewer false positives and faster execution.
- **Terrascan** reported the least findings, focusing more on compliance-based policies.

### Pulumi Security Analysis (KICS)

- Total findings: 6  
- HIGH: 2  
- MEDIUM: 1  
- CRITICAL: 1  

#### Key Issues

- Publicly accessible RDS instance (**CRITICAL**)
- Hardcoded secrets in configuration
- Missing encryption for DynamoDB
- Disabled monitoring for EC2

### Terraform vs Pulumi

- Terraform (HCL) exposed more misconfigurations due to declarative structure.
- Pulumi (YAML) showed fewer but more critical issues.
- Pulumi vulnerabilities were more related to **logic and configuration choices**, while Terraform had more **policy violations**.

### Tool Strengths

| Tool       | Strength |
|------------|----------|
| tfsec      | Fast, low false positives |
| Checkov    | Broad policy coverage |
| Terrascan  | Compliance-focused scanning |
| KICS       | Multi-framework support (Pulumi, Ansible) |



## Task 2 — Ansible Security Analysis (KICS)

- Total findings: 10  
- HIGH: 9  
- LOW: 1  

### Key Issues

- Hardcoded passwords in playbooks
- Secrets stored in inventory files
- Credentials exposed in URLs
- Use of root user with passwords

### Best Practice Violations

1. **Hardcoded Secrets**
   - Risk: Credential leakage
   - Fix: Use Ansible Vault

2. **No Secret Masking**
   - Risk: Logs expose sensitive data
   - Fix: Use `no_log: true`

3. **Insecure Authentication**
   - Risk: Unauthorized access
   - Fix: Use SSH keys instead of passwords

### Remediation

- Store secrets in **Ansible Vault**
- Replace passwords with **SSH key authentication**
- Avoid plaintext credentials in inventory files



## Task 3 — Comparative Tool Analysis

###  Tool Effectiveness Matrix
| Criterion | tfsec | Checkov | Terrascan | KICS |
|-----------|-------|---------|-----------|------|
| **Total Findings** | 53 | 78 | 22 | 16 |
| **Scan Speed** | Fast | Medium | Medium | Medium |
| **False Positives** | Low | Medium | Low | Medium |
| **Report Quality** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Ease of Use** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Documentation** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Platform Support** | Terraform only | Multiple | Multiple | Multiple |
| **Output Formats** | JSON, text | JSON, CLI, SARIF | JSON, human | JSON, HTML, CLI |
| **CI/CD Integration** | Easy | Easy | Medium | Medium |
| **Unique Strengths** | Speed | Coverage | Compliance | Multi-framework |

### Vulnerability Category Analysis
| Security Category | tfsec | Checkov | Terrascan | KICS (Pulumi) | KICS (Ansible) | Best Tool |
|------------------|-------|---------|-----------|---------------|----------------|----------|
| **Encryption Issues** | Good | Strong | Medium | Strong | N/A | Checkov |
| **Network Security** | Strong | Strong | Medium | Medium | Medium | Checkov |
| **Secrets Management** | Weak | Medium | Weak | Strong | Strong | KICS |
| **IAM/Permissions** | Medium | Strong | Medium | Medium | N/A | Checkov |
| **Access Control** | Strong | Strong | Medium | Medium | Medium | Checkov |
| **Compliance/Best Practices** | Medium | Strong | Strong | Medium | Medium | Terrascan |


### Tool Comparison Summary

| Tool        | Findings |
|-------------|---------|
| tfsec       | 53 |
| Checkov     | 78 |
| Terrascan   | 22 |
| KICS Pulumi | 6 |
| KICS Ansible| 10 |

### Category Analysis

| Category             | Best Tool |
|---------------------|----------|
| Secrets Detection   | KICS |
| IAM/Permissions     | Checkov |
| Encryption Issues   | tfsec |
| Compliance         | Terrascan |
| Multi-framework     | KICS |

### Top 5 Critical Findings

1. Public RDS instance (Pulumi)
2. Hardcoded database password
3. Unencrypted DynamoDB table
4. Open security groups (0.0.0.0/0)
5. Secrets in Ansible inventory

### Tool Selection Guide

- Use **tfsec** for fast CI/CD scans
- Use **Checkov** for comprehensive Terraform analysis
- Use **KICS** for Pulumi and Ansible
- Use **Terrascan** for compliance validation

### CI/CD Strategy

1. Run tfsec in pre-commit
2. Run Checkov in CI pipeline
3. Run KICS for multi-framework scanning
4. Combine results for full coverage

### Justification — Tool Selection & Strategy

The tool selection is based on combining **speed, coverage, and multi-framework support**.

- **tfsec** was chosen for fast Terraform scanning with low false positives, making it ideal for pre-commit and early-stage checks.
- **Checkov** provides the most comprehensive policy coverage, detecting the highest number of issues (78 findings), which makes it suitable for CI pipelines.
- **Terrascan** focuses on compliance and policy-as-code using OPA, making it useful for regulatory validation rather than deep vulnerability detection.
- **KICS** was selected for Pulumi and Ansible due to its strong multi-framework support and ability to detect secrets and misconfigurations across different IaC types.

The overall strategy is to **combine multiple tools** to maximize detection coverage and reduce blind spots, as no single tool identifies all vulnerabilities.


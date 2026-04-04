# Lab 7: Container Security

## Goal

The goal of this lab was to analyze container security using vulnerability scanning tools, configuration analysis, and runtime hardening techniques.

## Task 1: Container Image Scanning

### Top 5 Critical/High Vulnerabilities

| CVE / ID | Package | Severity | Impact |
|----------|--------|----------|--------|
| SNYK-JS-VM2-5772823 | vm2@3.9.17 | Critical | Remote Code Execution (RCE) allowing attackers to execute arbitrary code |
| SNYK-JS-VM2-5772825 | vm2@3.9.17 | Critical | Remote Code Execution via sandbox escape |
| SNYK-JS-VM2-5537100 | vm2@3.9.17 | Critical | Sandbox bypass leading to privilege escalation |
| SNYK-JS-VM2-15116160 | vm2@3.9.17 | High | Improper resource control leading to potential DoS |
| System packages (multiple CVEs) | OS dependencies | High | Various vulnerabilities affecting system libraries |

### Dockle Configuration Findings

No FATAL or WARN issues were reported. However, several INFO-level findings indicate security weaknesses:

- Missing HEALTHCHECK instruction  
  → reduces observability and makes it harder to detect compromised or unhealthy containers  

- Docker Content Trust is not enabled  
  → images are not verified, increasing risk of supply chain attacks  

- Presence of unnecessary files (.DS_Store)  
  → increases attack surface and may expose unintended data  


### Security Posture Assessment

**Does the image run as root?**  
Yes, the container runs as root by default, which increases security risks.


### Security Recommendations

- Run container as a non-root user (`USER` in Dockerfile)  
- Enable Docker Content Trust for image verification  
- Add HEALTHCHECK instruction to monitor container health  
- Remove unnecessary files from the image  
- Regularly update dependencies to fix known CVEs  
- Use minimal base images to reduce attack surface  



## Task 2: Docker Benchmark Analysis

### Summary Statistics

Based on Docker Bench Security results:

- PASS: ~15  
- WARN: ~10  
- FAIL: 0  
- INFO/NOTE: multiple  

### Analysis of Findings

No FAIL-level issues were detected. However, multiple WARN findings indicate security weaknesses:

#### 1. Content Trust Not Enabled (4.5)
- **Issue:** Docker Content Trust is disabled  
- **Impact:** Images are not verified, which increases the risk of pulling compromised or malicious images  
- **Remediation:**  
  ```bash
  export DOCKER_CONTENT_TRUST=1
  ```

## Task 3: Deployment Security Configuration Analysis

### 1. Configuration Comparison Table

| Configuration | Capabilities | Security Options | Memory | CPU | PIDs | Restart |
|--------------|-------------|------------------|--------|-----|------|---------|
| Default      | None        | None             | Unlimited | Unlimited | None | no |
| Hardened     | Drop ALL    | no-new-privileges | 512MB | 1 CPU | None | no |
| Production   | Drop ALL, Add NET_BIND_SERVICE | no-new-privileges | 512MB | 1 CPU | 100 | on-failure |

The table is based on actual `docker inspect` output.



### 2. Security Measure Analysis

#### a) `--cap-drop=ALL` and `--cap-add=NET_BIND_SERVICE`

- **Linux capabilities** are fine-grained permissions that control what a process can do (e.g., bind ports, mount filesystems, etc.)

- **Dropping ALL capabilities** removes all elevated privileges from the container, reducing attack surface and preventing privilege escalation.

- **Why add NET_BIND_SERVICE:**  
  Required to bind to low ports (<1024), which some applications may need.

- **Security trade-off:**  
  Minimal required privileges are restored, balancing security and functionality.



#### b) `--security-opt=no-new-privileges`

- Prevents processes from gaining additional privileges (e.g., via setuid binaries)

- **Prevents:** privilege escalation attacks

- **Downside:**  
  Some applications that rely on privilege escalation may fail



#### c) `--memory=512m` and `--cpus=1.0`

- Without limits, containers can consume all host resources

- **Prevents:** Denial of Service (DoS) attacks via resource exhaustion

- **Risk:**  
  Too strict limits may cause application crashes or degraded performance



#### d) `--pids-limit=100`

- A **fork bomb** is an attack where processes continuously spawn new processes

- PID limiting restricts number of processes → prevents system exhaustion

- **Choosing limit:**  
  Depends on application requirements; must balance functionality and security



#### e) `--restart=on-failure:3`

- Restarts container up to 3 times if it crashes

- **Useful:** improves availability and resilience

- **Risk:**  
  Can mask underlying issues or create restart loops

- **Comparison:**  
  - `on-failure` → controlled restart  
  - `always` → restarts regardless of cause (riskier)



### 3. Critical Thinking Questions

#### Which profile for DEVELOPMENT?

**Default configuration**

- Easier debugging  
- No restrictions  
- Faster iteration  



#### Which profile for PRODUCTION?

**Production configuration**

- Minimal privileges  
- Resource limits applied  
- Process limits enforced  
- Better fault tolerance  



#### What real-world problem do resource limits solve?

They prevent resource exhaustion attacks (DoS), where a container consumes all CPU or memory, affecting other services.



#### If an attacker exploits Default vs Production, what actions are blocked?

In Production:
- Cannot escalate privileges  
- Cannot spawn unlimited processes  
- Cannot consume unlimited resources  
- Limited system interaction due to dropped capabilities  



#### What additional hardening would you add?

- Run container as non-root user  
- Enable read-only filesystem (`--read-only`)  
- Use seccomp and AppArmor profiles  
- Enable network segmentation  
- Use image signing (Docker Content Trust)  

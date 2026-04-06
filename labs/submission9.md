### Task 1 — Runtime Security Detection with Falco

#### Baseline Alerts (Falco)

Falco was deployed using a containerized setup with the no-driver (userspace) configuration due to macOS limitations. Runtime security events were successfully captured using the Falco event generator.

The following baseline alerts were observed:

* **Create Symlink Over Sensitive Files**
  Detected creation of a symbolic link pointing to `/etc`, which is a sensitive directory. This behavior may indicate attempts to manipulate or expose critical system files.

* **Clear Log Activities**
  Detected access to log files suggesting potential log tampering. This behavior is often associated with attempts to hide malicious activity.

* **Remove Bulk Data from Disk**
  Detected execution of a command (`shred`) used to delete data securely. This may indicate destructive behavior or anti-forensics.

* **Find AWS Credentials**
  Detected attempts to search for AWS credentials in the filesystem. This is a common indicator of credential harvesting activity.

* **Disallowed SSH Connection (Non-standard port)**
  Detected an SSH connection attempt over port 443. This may indicate attempts to bypass firewall restrictions or establish covert communication channels.

All alerts were recorded in:

```
labs/lab9/analysis/falco-baseline.log
```

These alerts demonstrate Falco’s ability to detect suspicious runtime behavior across filesystem, process, and network activity.

---

#### Custom Rule

A custom Falco rule was implemented to detect writes to `/usr/local/bin`, a directory typically used for executable binaries.

```yaml
- rule: Write Binary Under UsrLocalBin
  desc: Detect writes under /usr/local/bin inside any container
  condition: container.id != host and fd.name startswith /usr/local/bin/
  output: >
    Falco Custom: File write in /usr/local/bin (container=%container.name file=%fd.name)
  priority: WARNING
  tags: [container, drift]
```

**Purpose:**
The rule is designed to detect potential container drift or unauthorized modification of executable paths. Writing files to `/usr/local/bin` at runtime may indicate:

* malicious binary injection
* privilege escalation attempts
* post-exploitation persistence mechanisms

**When it should fire:**

* when a container process creates or modifies files under `/usr/local/bin`
* especially when performed by privileged users (e.g., root)

**When it should NOT fire:**

* during normal container startup if binaries are part of the image
* in read-only or properly hardened containers
* when no runtime modifications occur in binary directories

The rule was successfully validated by executing:

```bash
docker exec --user 0 lab9-helper /bin/sh -lc 'echo custom-test > /usr/local/bin/custom-rule.txt'
```

This triggered the alert:

```
Falco Custom: File write in /usr/local/bin
```

Evidence is available in:

```
labs/lab9/analysis/falco-custom.log
```

---

### Task 2 — Policy-as-Code with Conftest

#### Policy Violations in Unhardened Manifest

The `juice-unhardened.yaml` manifest triggered multiple policy violations, indicating missing security best practices:

* **Missing resource limits and requests (CPU, memory)**
  Containers without resource constraints can consume excessive system resources, leading to Denial-of-Service (DoS) conditions and unstable cluster behavior.

* **Missing `allowPrivilegeEscalation: false`**
  Without this restriction, processes inside the container may gain elevated privileges, increasing the risk of container breakout or privilege escalation attacks.

* **Missing `readOnlyRootFilesystem: true`**
  A writable root filesystem allows attackers to modify binaries or inject malicious files, enabling persistence inside the container.

* **Missing `runAsNonRoot: true`**
  Running as root increases the impact of a compromise. Attackers gaining access to a root container have more control over the system.

* **Use of `:latest` image tag**
  The `latest` tag is not deterministic and may introduce unverified or vulnerable image versions, breaking reproducibility and security guarantees.

* **Missing livenessProbe and readinessProbe (warnings)**
  Without health checks, Kubernetes cannot detect unhealthy containers, which affects availability and reliability.

Evidence:

```
labs/lab9/analysis/conftest-unhardened.txt
```

---

#### Hardening Improvements in Hardened Manifest

The `juice-hardened.yaml` manifest resolves all detected issues and passes all policy checks:

* **Resource constraints added**
  Defined CPU and memory requests/limits ensure predictable resource usage and prevent abuse.

* **Privilege escalation disabled**
  `allowPrivilegeEscalation: false` prevents processes from gaining higher privileges.

* **Read-only root filesystem enabled**
  `readOnlyRootFilesystem: true` prevents runtime modification of critical files, reducing persistence risks.

* **Non-root execution enforced**
  `runAsNonRoot: true` ensures the container runs with limited privileges.

* **Stable image tag used**
  Replacing `:latest` with a fixed version improves reproducibility and security.

* **Health checks configured**
  Liveness and readiness probes improve reliability and allow Kubernetes to manage container lifecycle effectively.

Result:

```
30 tests, 30 passed, 0 warnings, 0 failures
```

Evidence:

```
labs/lab9/analysis/conftest-hardened.txt
```

---

#### Docker Compose Policy Analysis

The Docker Compose manifest was evaluated using Conftest policies defined in `compose-security.rego`.

Key aspects analyzed:

* **Container privilege settings**
  Ensuring containers do not run in privileged mode or with excessive capabilities.

* **Image configuration**
  Avoiding the use of `latest` tags and encouraging pinned versions.

* **Security options**
  Validating the presence of configurations such as read-only filesystems or dropped capabilities where applicable.

* **Resource configuration (if defined)**
  Checking whether containers define limits to prevent resource exhaustion.

These policies help enforce consistent security standards across non-Kubernetes environments, ensuring that even local or development deployments follow best practices.


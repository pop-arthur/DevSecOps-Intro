# Lab 3

## Task 1 — SSH Commit Signature Verification

### Summary explaining the benefits of signing commits for security
Commit signing provides a cryptographic way to verify the authenticity and integrity of commits. It ensures that commits are created by a trusted developer and have not been modified after being signed.

The main security benefits include:

- **Authentication** — verifies the identity of the author, preventing impersonation
- **Integrity** — guarantees that the commit content has not been altered
- **Non-repudiation** — the author cannot deny creating the commit
- **Protection against supply chain attacks** — ensures that malicious actors cannot inject unauthorized changes into the repository
- **Trust in collaboration** — team members and CI/CD systems can trust that commits come from verified sources

In DevSecOps workflows, commit signing is essential because it helps secure the software development lifecycle by ensuring that only verified and trusted code is integrated into the system.


### Evidence of successful SSH key setup and configuration
```
arthur@Artur-MacBook-Pro DevSecOps-Intro % git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgSign true
git config --global gpg.format ssh
arthur@Artur-MacBook-Pro DevSecOps-Intro % git config --global user.email
ar.popov@innopolis.university
arthur@Artur-MacBook-Pro DevSecOps-Intro % git config --global user.email "artu.200505@gmail.com" 
arthur@Artur-MacBook-Pro DevSecOps-Intro % git commit -S -m "docs: add commit signing summary"
[feature/lab3 435e0b8] docs: add commit signing summary
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 labs/submission3.md
 
arthur@Artur-MacBook-Pro DevSecOps-Intro % git push --set-upstream origin feature/lab3
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 665 bytes | 665.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
remote: 
remote: Create a pull request for 'feature/lab3' on GitHub by visiting:
remote:      https://github.com/pop-arthur/DevSecOps-Intro/pull/new/feature/lab3
remote: 
To github.com:pop-arthur/DevSecOps-Intro.git
 * [new branch]      feature/lab3 -> feature/lab3
branch 'feature/lab3' set up to track 'origin/feature/lab3'.
```

### Analysis: Why is commit signing critical in DevSecOps workflows?

Commit signing is critical in DevSecOps workflows because it provides a reliable and secure way to verify the authenticity and integrity of code changes. It ensures that the code in the repository is trusted and has not been tampered with, protecting against supply chain attacks and impersonation attacks. This trust is crucial for secure software development, as it enables the use of automated tools and processes that rely on verified code. Additionally, commit signing promotes collaboration and trust within development teams, as it provides a way to verify the identity of the authors of the code.

### Screenshots or verification of the "Verified" badge on GitHub
![img.png](screenshots/lab03/img.png)


## Task 2 — Pre-commit Secret Scanning 

### Pre-commit hook setup process and configuration
```
arthur@Artur-MacBook-Pro DevSecOps-Intro % cd .git/hooks 
arthur@Artur-MacBook-Pro hooks % nano pre-commit
arthur@Artur-MacBook-Pro hooks % cd ../..
arthur@Artur-MacBook-Pro DevSecOps-Intro % chmod +x .git/hooks/pre-commit
```

### Test results showing both blocked and successful commits

Blocked:
```
arthur@Artur-MacBook-Pro DevSecOps-Intro % echo "password=MySuperSecretPassword123" > test.txt

arthur@Artur-MacBook-Pro DevSecOps-Intro % git add test.txt                                   

arthur@Artur-MacBook-Pro DevSecOps-Intro % git commit -m "test secret"                        
[pre-commit] scanning staged files for secrets…
[pre-commit] Files to scan: test.txt
[pre-commit] Non-lectures files: test.txt
[pre-commit] Lectures files: none
[pre-commit] TruffleHog scan on non-lectures files…
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-04-03T10:25:17Z    info-0  trufflehog      running source  {"source_manager_worker_id": "MBfQi", "with_units": true}
2026-04-03T10:25:17Z    info-0  trufflehog      finished scanning       {"chunks": 1, "bytes": 34, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "2.091542ms", "trufflehog_version": "3.94.2", "verification_caching": {"Hits":0,"Misses":0,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
[pre-commit] ✓ TruffleHog found no secrets in non-lectures files
[pre-commit] Gitleaks scan on staged files…
[pre-commit] Scanning test.txt with Gitleaks...
[pre-commit] No secrets found in test.txt

[pre-commit] === SCAN SUMMARY ===
TruffleHog found secrets in non-lectures files: false
Gitleaks found secrets in non-lectures files: false
Gitleaks found secrets in lectures files: false

✓ No secrets detected in non-excluded files; proceeding with commit.
[feature/lab3 56f87bf] test secret
 1 file changed, 1 insertion(+), 2 deletions(-)
arthur@Artur-MacBook-Pro DevSecOps-Intro % echo "anything" > test.txt
git add test.txt
git commit -m "test secret"
[pre-commit] scanning staged files for secrets…
[pre-commit] Files to scan: test.txt
[pre-commit] Non-lectures files: test.txt
[pre-commit] Lectures files: none
[pre-commit] TruffleHog scan on non-lectures files…
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-04-03T10:29:09Z    info-0  trufflehog      running source  {"source_manager_worker_id": "LZUtY", "with_units": true}
2026-04-03T10:29:09Z    info-0  trufflehog      finished scanning       {"chunks": 1, "bytes": 9, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "1.957333ms", "trufflehog_version": "3.94.2", "verification_caching": {"Hits":0,"Misses":0,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
[pre-commit] ✓ TruffleHog found no secrets in non-lectures files
[pre-commit] Gitleaks scan on staged files…
[pre-commit] Scanning test.txt with Gitleaks...
Gitleaks found secrets in test.txt:
10:29AM INF scanned ~9 bytes (9 bytes) in 47.8ms
10:29AM INF no leaks found
---
✖ Secrets found in non-excluded file: test.txt

[pre-commit] === SCAN SUMMARY ===
TruffleHog found secrets in non-lectures files: false
Gitleaks found secrets in non-lectures files: true
Gitleaks found secrets in lectures files: false

✖ COMMIT BLOCKED: Secrets detected in non-excluded files.
Fix or unstage the offending files and try again.
```

Successful:
```
arthur@Artur-MacBook-Pro DevSecOps-Intro % echo "anything" > test.txt 
git add test.txt
git commit -m "test secret"
[pre-commit] scanning staged files for secrets…
[pre-commit] Files to scan: test.txt
[pre-commit] Non-lectures files: test.txt
[pre-commit] Lectures files: none
[pre-commit] TruffleHog scan on non-lectures files…
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-04-03T10:57:16Z    info-0  trufflehog      running source  {"source_manager_worker_id": "m2YXJ", "with_units": true}
2026-04-03T10:57:16Z    info-0  trufflehog      finished scanning       {"chunks": 1, "bytes": 9, "verified_secrets": 0, "unverified_secrets": 0, "scan_duration": "3.312583ms", "trufflehog_version": "3.94.2", "verification_caching": {"Hits":0,"Misses":0,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}}
[pre-commit] ✓ TruffleHog found no secrets in non-lectures files
[pre-commit] Gitleaks scan on staged files…
[pre-commit] Scanning test.txt with Gitleaks...
[pre-commit] No secrets found in test.txt

[pre-commit] === SCAN SUMMARY ===
TruffleHog found secrets in non-lectures files: false
Gitleaks found secrets in non-lectures files: false
Gitleaks found secrets in lectures files: false

✓ No secrets detected in non-excluded files; proceeding with commit.
[feature/lab3 50780c1] test secret
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
```

### Analysis

Automated secret scanning helps prevent security incidents by detecting sensitive data before it is committed to a repository.

It prevents accidental leakage of credentials such as API keys and passwords, stopping them from entering Git history, where they are difficult to remove. This reduces the risk of unauthorized access and potential breaches.

Additionally, it enforces a shift-left security approach by integrating security checks directly into the development workflow. As a result, developers become more aware of handling sensitive data securely.

Overall, automated scanning acts as a proactive security measure, preventing incidents before they occur rather than fixing them later.
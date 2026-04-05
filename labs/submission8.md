# Lab 8 — Software Supply Chain Security: Signing, Verification, and Attestations

## Overview

This lab demonstrates container image signing, verification, tamper detection, and attestations (SBOM and provenance), as well as signing of non-container artifacts.

# Task 1 — Signing & Verification

## Image Push & Digest Reference

Image was pushed to a local registry and referenced by immutable digest:

```
localhost:5001/juice-shop@sha256:872efcc03cc16e8c4e2377202117a218be83aa1d05eb22297b248a325b400bd7
```

## Signing

The image was signed using Cosign with a locally generated key pair.

## Verification

Signature verification succeeded:

* Signature is valid
* Digest matches signed content
* Public key verification passed

## Tamper Demonstration

The tag `v19.0.0` was overwritten with a different image (`busybox`).

New digest:

```
sha256:50a3a2fef78c92dee45a3a9b72af5bdcbff6476e685cef49d97f286b6ce6f14a
```

Verification result:

* `no signatures found`

## Key Insight

* Tags are mutable → can be overwritten
* Digests are immutable → guarantee integrity
* Signing binds trust to the digest, not the tag

## To document

**How signing protects against tag tampering**

Container image signing protects against tag tampering by binding the signature to the image’s immutable digest rather than its tag. Tags are mutable and can be reassigned to different images, allowing an attacker to replace the content behind a tag without changing its name. However, when an image is signed, the signature is linked to a specific digest (hash of the image). If the tag is later overwritten with a different image, its digest changes, and the existing signature no longer matches. As a result, signature verification fails, exposing the tampering.


**What “subject digest” means**

The subject digest is the cryptographic hash (e.g., SHA256) of a container image that uniquely identifies its exact content. It acts as an immutable fingerprint: any change to the image results in a completely different digest. In signing and attestations, the subject digest is used as the trusted reference to ensure integrity, reproducibility, and authenticity of the image.

# Task 2 — Attestations

## SBOM Attestation (CycloneDX)

* SBOM generated using Syft
* Converted to CycloneDX format
* Attached using Cosign attestation

Verification:

* Attestation successfully validated
* Payload extracted and decoded

SBOM contains:

* List of dependencies
* Versions of packages
* Software composition details

## Provenance Attestation

A minimal SLSA provenance predicate was created and signed.

Contains:

* Builder identity (`student@local`)
* Build timestamp
* Input parameters (image digest)

Verification:

* Attestation is valid
* Payload confirms build metadata

## Key Insight

* Signature = verifies *what* was deployed
* Attestation = explains *how* it was built

## To document

**How attestations differ from signatures**

Signatures verify the integrity and authenticity of an artifact by proving that it was signed by a trusted key and has not been modified. They answer the question *“Is this artifact trustworthy?”*.
Attestations, on the other hand, provide additional metadata about the artifact. They answer questions like *“What is inside the artifact?”* or *“How was it built?”*. While signatures ensure integrity, attestations provide context and transparency about the software supply chain.


**What information the SBOM attestation contains**

An SBOM (Software Bill of Materials) attestation contains a structured list of all components included in an artifact. This typically includes:

* Names of dependencies and packages
* Versions of each component
* Licensing information
* Relationships between components

This information allows security teams to identify vulnerabilities, track dependencies, and ensure compliance with licensing and security policies.


**What provenance attestations provide for supply chain security**

Provenance attestations provide information about how an artifact was built and by whom. They typically include:

* Builder identity
* Build process and environment
* Input parameters and source references
* Timestamps of the build

This helps establish trust in the software supply chain by enabling traceability, reproducibility, and verification of the build process. It ensures that the artifact was produced in a controlled and trusted environment, reducing the risk of supply chain attacks.


# Task 3 — Artifact Signing

A non-container artifact (`sample.tar.gz`) was created and signed.

## Signing

* Used `cosign sign-blob`
* Bundle file generated

## Verification

```
Verified OK
```

## Use Cases

* Signing release binaries
* Verifying config files
* Protecting distributed artifacts

## Difference from Image Signing

* Image signing -> stored in registry
* Blob signing -> standalone files with bundle

## To document

**Use cases for signing non-container artifacts**

Blob signing is used to protect standalone files such as release binaries, configuration files, scripts, and archives. It ensures these artifacts are authentic and have not been tampered with before distribution or deployment.

**How blob signing differs from container image signing**

Blob signing applies to individual files and uses local signature bundles, while container image signing targets images stored in registries and attaches signatures to their digests.

# Verifying Kubernetes Platform Binaries

## Why to verify the K8s platform binaries?

- **Prevents Tampering (Integrity)**
  - The primary reasons for this verification are to prevent supply chain attacks and ensure that the software has not been tampered with or corrupted during transit.
- **Ensures Authenticity (Supply Chain Security)**
  - It ensures that the binaries originate from the legitimate and trusted source (the Kubernetes project) and not an imposter

- **Mitigates Malicious Code Execution**
  - If a binary file is compromised, running it could give a malicious user control over your cluster components (API server, kubelet, etc.), leading to a full system compromise

- **Aids in Regulatory Compliance**
  - Many industry security standards and benchmarks, such as the Center for Internet Security (CIS) benchmarks for Kubernetes, recommend verifying binary integrity as part of a secure cluster setup.

## How the Binary Verification works?

The binary verification process involves using the _cryptographic hash (checksum)_ files provided by the Kubernetes release team.

- **Download the binaries with the checksums (SHA-512)**
  - You download the Kubernetes binaries (e.g., kube-apiserver, kubelet, kubectl) and their corresponding checksum file (often a SHA512 hash) from the official release page.
- **Generate a Checksum**
  - You use a local tool (like sha512sum) to generate the hash of the downloaded binary file on your system.
- **Compare**
  - You compare the locally generated hash with the official hash provided by the Kubernetes project.
- **Confirm**
  - If the hashes match, the binary is authentic and untampered. A mismatch indicates a potential security risk, and the file should be deleted and re-downloaded.

## Common Hash Functions

- MD5 (Message Digest Algorithm 5): Produces a 128-bit hash value. It is fast but considered cryptographically broken and unsuitable for further use.
- SHA-1 (Secure Hash Algorithm 1): Produces a 160-bit hash value. It is also considered weak due to vulnerabilities to collision attacks.
- SHA-256 (Secure Hash Algorithm 256-bit): Part of the SHA-2 family, it produces a 256-bit hash value and is widely used for its strong security properties.
- SHA-3: The latest member of the Secure Hash Algorithm family, designed to provide a higher level of security.

## Hands-on Lab: Verify Kubernetes platform binaries

- Download the specific Kubernetes binaries you need (e.g., kube-apiserver, kubectl, kubelet) from the official Kubernetes GitHub release page or the Kubernetes release downloads page.
- Locate and download the corresponding CHANGELOG file or the separate checksum file (often with a .sha512 or similar extension) for that specific version and architecture into the same directory as the binaries

```
# Download the Binary
curl https://dl.k8s.io/v1.31.0/kubernetes.tar.gz -L -o kubernetes.tar.gz

# Generate the Checksum - on Linux and MacOS
shasum -a 512 kubernetes.tar.gz
OR
# Generate the Checksum - on Linux
sha512sum kubernetes.tar.gz
```

- Navigate to the K8s Release page and ensure that the output of the chosen checksum command exactly matches the hash available on the release page.
- A mismatch may indicate that the file has been tampered with.

## References

- Kubernetes Releases (https://kubernetes.io/releases/)
- Kubernetes 1.34 Release Notes (https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.34.md)

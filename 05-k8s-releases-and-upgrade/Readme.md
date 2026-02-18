# Kubernetes Releases and Upgrade

## 01. Kubernetes Releases

### Overview

- K8s maintains a steady release cycle for its minor versions, typically delivering _three major updates per year_.
- Each release follows a strict support window of approximately 14 months (12 months of standard support followed by 2 months of maintenance).
- For **latest Kubernetes Releases** - https://kubernetes.io/releases/

- [Kubernetes Patch Releases](https://kubernetes.io/releases/patch-releases/)

- [Upcoming Monthly Releases](https://kubernetes.io/releases/patch-releases/#upcoming-monthly-releases)

- [Detailed Release History for Active Branches](https://kubernetes.io/releases/patch-releases/#detailed-release-history-for-active-branches)

### Kubernetes Versions Structure

- Kubernetes versions follow a specific structure based on Semantic Versioning (SemVer) to communicate the impact of changes.

- Versions are written in the format **`vX.Y.Z`**
  - **Major Version (X)**
    - Represents significant, potentially breaking changes to the core platform or API.
    - K8s has remained on v1 since its first stable release in 2015.
  - **Minor Version (Y)**
    - These are the primary feature releases:
      - **Cadence**: Released roughly 3 times per year (every 4 months) | e.g., 1.30 to 1.31 | Patch Releases: Monthly | Support Window: three most recent minor releases

      - **Contents**: Includes new features, enhancements, and API deprecations.
      - **Support**: The community officially supports the three most recent minor versions (N-2).

  - **Patch Version (Z)**
    - Focuses on backward-compatible bug fixes and security updates.
    - Cadence: Released frequently (often weekly) for active minor versions

### Download the Kubernetes Package

- You may download the kubernetes from official Github repo - https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/

- The downloaded package when extracted will have all the control plane components in it, all will be of the same version:
  - kube-apiserver
  - controller-manager
  - kube-scheduler
  - kubelet
  - kube-proxy
  - kubectl

- Control plane components that do not have the same version as above control plane components:
  - etcd
  - CodeDNS

## 02. Kubernetes Upgrade

### 2.1 Recommended versions for K8s Components

### 2.2 How the Kubernetes upgrade works?

## 03. Reference

- https://kubernetes.io/releases/
- https://github.com/kubernetes/kubernetes/releases

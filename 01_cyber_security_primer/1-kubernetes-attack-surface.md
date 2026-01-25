# Kubernetes Attack Surface

The K8s attack surface encompasses all potential entry points and vulnerabilities that an attacker could exploit to gain unauthorized access to a cluster.

## 4 C's of Cloud Native Security

1. **Cloud/Infrastructure**
   - The underlying physical or cloud infrastructure where the cluster runs
2. **Cluster**
   - etcd
   - worker nodes
   - api-server
   - Pod Security Standards (PSS)

- CKS Exam Scope: Hardening the cluster, managing API access, and network segmentation.

3. **Container**
   - Securing container images and runtime environments
   - Image scanning (vulnerabilities), minimal base images, runtime security (Seccomp, AppArmor), secrets management, resource limits

- CKS Exam Scope: Image signing, vulnerability management, secure container lifecycle.

4. **Code**
   - The application code itself, including third-party libraries and dependencies
   - Secure Software Development Lifecycle (SSDLC), dependency scanning, supply chain security.

## Kubernetes Common Attack Vectors

- **Unauthorized Access OR API Misconfiguration**

  - Attackers can exploit exposed sensitive interfaces like the K8s Dashboard or API server.
  - Unsecured API endpoints allow direct control over the cluster, leading to unauthorized access, injection, or DoS.

- **Privilege Escalation**

  - Exploiting misconfigured container permissions to gain higher privileges within the cluster or access the host system.
  - Over permissive roles grant attackers broad access, enabling privilege escalation and lateral movement.

- **Malicious Containers** OR **Supply Chain Attacks**

  - Deploying compromised or malicious containers
  - Malicious code injected into container images or third-party dependencies.

- **Denial of Service (DoS)**

  - Attacking critical components like the API server or CoreDNS to disrupt cluster operations.

- **Lateral Movement within cluster**
  - Once a foothold is established, an attacker can move across pods and nodes. It is often occurs due to poorly defined network policies that allow unrestricted pod-to-pod communication by default.

## Recommended Mitigation Strategies

- Least Privilege Principle

  - Enforce strict RBAC and Network Policies

- Image Security

  - Scan images for vulnerabilities and use trusted registries

- Secure Configurations

  - Harden API server, control access to etcd, and secure Kubelet

- Monitoring & Auditing

  - Continuously audit permissions and monitor network traffic for anomalies

- Defense-in-Depth
  - Implement layered security controls, including runtime security and admission controllers

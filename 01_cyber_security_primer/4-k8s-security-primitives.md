# Kubernetes Security Primitives

- Kubernetes security primitives are the foundational, built-in mechanisms used to protect clusters, workloads, and data.
- They are often categorized by how they secure access to the API, isolate workloads, and protect internal communications.

1. API Server Access Control

   - Authentication (AuthN)

     - Verifies the identity of a user or service account. Methods include X.509 client certificates, Static Tokens, and integration with external OIDC (OpenID Connect) providers.

   - Authorization (AuthZ)

     - RBAC (Role-based Access Control)
     - ABAC (Attribute-based Access Control)
     - Node Authorization

   - Addmission Control

2. Workload & Container Security

   - **Security Context**

     - A set of fields in a Pod or Container spec that defines privilege and access control settings, such as _runAsUser_, _runAsNonRoot_, and _readOnlyRootFilesystem_

   - Pod Security Admission (PSA)
   - Secrets
     - Objects used to store sensitive information like passwords or keys.
     - While stored as Base64, they should be combined with Encryption at Rest in the etcd database for true security.
   - Resource Quotas & LimitRanges
     - Primitives that prevent "Denial of Service" (DoS) attacks from within the cluster by limiting the CPU and memory a namespace or pod can consume

3. Network Security

   - As we already know that by default, all pods in a cluster can communicate with each other.
   - Network Policies
     - The primary primitive for defining how groups of pods are allowed to communicate with each other and other network endpoints.
     - They function as a distributed firewall

4. Infrastructure & Data Protection
   - Transport Layer Security (TLS)
     - Kubernetes expects all internal communication (e.g., between the API server and Kubelet, or within the etcd cluster) to be encrypted via TLS
   - Auditing
     - Provides a chronological set of records documenting every action taken in the cluster for forensic review
   - Etcd Security
     - The cluster's "brain" must be isolated behind firewalls and secured with mutual TLS (mTLS) to prevent unauthorized state changes

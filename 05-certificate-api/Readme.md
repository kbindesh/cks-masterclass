# Kubernetes TLS Certificates and Certificate Signing Requests (CSR)

In this section, you will learn how Kubernetes Certificates API works. It covers the following concepts:

- Create Private key (.key)
- Create kubernetes _Certificate Signing Request (CSR)_ object
- Review CSR requests
- Approve/Reject CSR request
- Share the Signed certificate with users

## Hands-on Lab: Onboard a new user on K8s cluster using TLS Certificate based authN and approve the CSR using `Certificates API`

### Step-01: Generate private key for a User

```
# Generate a private key for a user, say developer
openssl genrsa -out developer.key 2048
```

### Step-02: Create a Certificate Signing Request (CSR) object

```
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer@binorg.com"
```

### Step-03: Approve/Reject Certificate Signing Request (CSR)

```

```

### Step-04: Share the signed certificate with the User

## Key Concepts

- In a K8s cluster, a Certificate Signing Request (CSR) is typically created by a user or service account who needs a client certificate to authenticate to the cluster.
- The creation process is usually handled programmatically by a controller or a command-line tool, rather than being manually created by an admin.
  - **Users or Service Accounts**
    - The entity requesting the certificate must have the necessary Role-Based Access Control (RBAC) permissions to create the CertificateSigningRequest resource.
    - They need the _create_ verb on the _certificates.k8s.io_ API group's certificatesigningrequests resource
  - **Automated Processes**
    - Tools like kubelet automatically create a CSR when a new node joins the cluster, requesting a certificate for itself.
    - Other common use cases involve service mesh systems (like Istio or Linkerd) and certificate controllers (like cert-manager) that automate the issuance of workload certificates.
  - **Cluster Administrators**
    - While administrators can manually create a CSR (often for bootstrapping or troubleshooting), the normal operational flow is designed for automated systems or end-users with limited privileges to initiate the request, which is then reviewed and approved by an authorized signer.

### Who can approve the CSR?

- **Cluster Administrators**
  - The primary approvers, using kubectl commands to manually review and sign (or deny) CSRs.
- **Automated Controllers**
  - For scale, custom controllers (like the Kubelet CSR Approver) can automatically approve CSRs that pass configured checks, often integrating with the core controller manager.

### How CSR Approval works?

1. **Request** - A component (like a Kubelet or user) creates a CertificateSigningRequest (CSR) object.
2. **Pending** - The CSR enters a Pending state, waiting for approval.
3. **Approval/Denial** - An administrator or controller approves/denies the CSR.
4. **Signing** - The Kubernetes CA (or specified signer) signs the approved request, issuing a certificate.

### Commands for manually approving CSR

```
# Lists pending CSRs
kubectl get csr

# Approves the request
kubectl certificate approve <csr-name>

# Denies the request
kubectl certificate deny <csr-name>
```

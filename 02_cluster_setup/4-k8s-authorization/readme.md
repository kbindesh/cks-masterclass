# Kubernetes Authorization

## Lab: Setup User Authentication using TLS Certificates and Authorization using Role-Based-Access-Control (RBAC)

### Step-01: Lab Overview

- In this lab, you will learn how to provide a new User access to Kubernetes cluster using TLS-based Authentication and elevate access levels using RBAC.
  - We will create TLS Certificate for a Developer
  - Assign system:masters role to the user
  - Update the kubeconfig file with new user details
  - Access the kubernetes cluster as a cluster admin (custom user)

### Step-02: Prerequisites

- Kubernetes Cluster with atleast 1 worker node (with admin access)
- kubectl
- openssl

### Step-03: Key Considerations

In a kubeadm based kubernetes cluster, following are the key directories:

- **Cluster internal TLS Certificates, User Certificates on Control Node** - /etc/kubernetes/pki
- **kubeconfig file** - /etc/kubernetes/

- **kubelet and Node Certificates on Worker nodes** - /var/lib/kubelet/pki/

### Step-04: Switch to K8s default pki directory

```
cd /etc/kubernetes/pki
```

### Step-05: Generate TLS Certificate for a group of User (e.g. developers) and get it signed by CA

```
USER_NAME="dev1"

# Generate a Private Key - dev1.key
openssl genrsa -out $USER_NAME.key 2048

# Create an x509 Certificate Signing Request (CSR) - dev1.csr
openssl req -new -key $USER_NAME.key -subj "/CN=dev1/O=developers" -out $USER_NAME.csr

# Generate a CA-signed certificate for the user - dev1.crt
openssl x509 -req -in $USER_NAME.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -out $USER_NAME.crt

# Inspect/Describe the Certificate details | Observe the Common Name (CN) & Organization (O)
openssl x509 -in $USER_NAME.crt -text -noout
```

### Step-XX: Create a new Kubernetes Role for Developers

### Step-XX: Create a new Kubernetes RoleBinding for Developers Role

### Step-XX: Update the kubeconfig file with new User (developer) details

### Step-XX: Verify User's access to Kubernetes and access levels

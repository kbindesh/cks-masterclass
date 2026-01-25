# Kubernetes Authentication

## Lab: User Authentication in Kubernetes using TLS Certificates

### Step-01: Lab Overview

- In this lab, you will learn how to provide a new User access to Kubernetes cluster using TLS-based Authentication.
  - We will create TLS Certificate for Cluster Administrator
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

### Step-05: Generate TLS Certificate for a User and get it signed by CA

```
USER_NAME="cluster-admin"

# Generate a Private Key
openssl genrsa -out $USER_NAME.key 2048

# Create x509 Certificate Signing Request (CSR) for a User
openssl req -new -key $USER_NAME.key -subj "/CN=cluster-admin/O=system:masters" -out $USER_NAME.csr

# Generate a CA-signed certificate for the user
openssl x509 -req -in $USER_NAME.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -out $USER_NAME.crt

# Inspect the Certificate details
openssl x509 -in $USER_NAME.crt -text -noout
```

### Step-06: Update the kubeconfig file with new User details

- The _kubeconfig_ file is by default located in the home directory (~/.kube/config) in text format.

```
# (optional) To get the kube-api-server URL (e.g. https://10.0.0.10:6443)
kubectl config view --minify | grep server

# Add the new credentials to the kubeconfig file
kubectl config set-credentials $USER_NAME --client-certificate=$USER_NAME.crt --client-key=$USER_NAME.key --embed-certs=true

# Create a new context for the rbac-labuser user
kubectl config set-context $USER_NAME-context --cluster=kubernetes --user=$USER_NAME --namespace=default

# Switch to a particular context and verify the current-context
kubectl config use-context $USER_NAME-context
kubectl auth whoami

[The preceding command should respond as - cluster-admin]
```

- Now, as kubeconfig file is updated with the new user (cluster-admin) context, it's time to make a callout to kubernetes and verify user's access.

### Step-07: Verify User's access to Kubernetes

```
kubectl auth can-i get pods

kubectl get pods -A

[The preceding command should list all the existing Pods]
```

- **Conclusion**
  - The cluster-admin user can now authenticate and use kubectl commands.
  - The active namespace is default, avoiding the need to specify -n default
  - All interactions go through cluster-admin-context, ensuring the right cluster, user, and namespace are set.

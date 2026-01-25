# Kubernetes Authorization

## Lab: Onboard a new User using TLS Certificates-based authentication and Authorization using RBAC

### Step-01: Lab Overview

- In this lab, you will learn how to provide a new User access to Kubernetes cluster using TLS-based Authentication and elevate access levels using RBAC.
  - We will create TLS Certificate for a Developer
  - Create a kubernetes _Role_ and a _roleBinding_ object for developers (group of users)
  - Assign a custom role (developers) to the user
  - Update the kubeconfig file with new user details
  - Access the kubernetes cluster as a developer (custom user) and verify his access.

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

### Step-06: Create a new Kubernetes `Role` for Developers

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developers-role
  namespace: development
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "update", "delete"]
```

### Step-07: Create a new Kubernetes `roleBinding` for Developers Role

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rolebinding-to-dev-group # Name of the RoleBinding resource
  namespace: development # Namespace where this binding is effective
subjects:
  - kind: Group
    name: developers # Name of your group (as recognized by your auth provider)
    apiGroup: rbac.authorization.k8s.io # Required for Group subjects
roleRef:
  kind: Role # The type of role being referenced (can be 'Role' or 'ClusterRole')
  name: developers-role # Name of the Role or ClusterRole to bind
  apiGroup: rbac.authorization.k8s.io # API group the role belongs to
```

### Step-08: Update the kubeconfig file with new User (developer) details

```
# Configure the rbac-labuser User in Kubeconfig
kubectl config set-credentials $USER_NAME --client-certificate=$USER_NAME.crt --client-key=$USER_NAME.key --embed-certs=true

# Create a new context for the rbac-labuser user
kubectl config set-context $USER_NAME-context --cluster=kubernetes --user=$USER_NAME --namespace=default

# Switch to a particular context and test
kubectl config use-context $USER_NAME-context
kubectl auth whoami
```

### Step-09: Verify User's access to Kubernetes and access levels

```
kubectl auth can-i update pod -n development

kubectl auth can-i delete pod -n development

kubectl auth can-i get pod -n development
```

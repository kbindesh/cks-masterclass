# Kubernetes Authorization

## Authorization Modes

1. **AllowsAllow**
   - This mode allows all the requests.
   - You must use this authorization mode only if you do not require authorization for your API requests (for example, while testing)
2. **AlwaysDeny**
   - This mode blocks all requests.
   - You must use this authorization mode only for testing.

3. **ABAC (attribute-based access control)**

4. **RBAC (role-based access control)**
   - Uses the _rbac.authorization.k8s.io_ API group to drive authorization decisions.
   - RBAC regulates access to the K8s API and cluster resources based on the roles assigned to users, groups, or service accounts.
   - _Core RBAC API Resources_: _Role_, _ClusterRole_, _RoleBinding_, _ClusterRoleBinding_.

5. **Node Authorization**
   - It grants permissions to kubelets based on the pods they are scheduled to run.

6. **Webhook**

## RBAC - Built-in Resources

### Key Built-in Groups

- **system:masters**
  - The most powerful group. Members bypass all RBAC/webhook authorization checks and have full cluster-admin rights, often used in emergency scenarios.
- **system:authenticated**
  - Automatically includes any user that successfully authenticates with the cluster.
- **system:unauthenticated**
  - Represents any request that cannot be authenticated.
- **system:nodes**
  - Used by kubelets to authenticate and gain permission to access specific node-related resources.
- **system:serviceaccounts**
  - Includes all service accounts created in the cluster.

### Commonly used K8s System Roles

- **cluster-admin**
  - Superuser role, often bound to system:masters.
- **admin**
  - Administrator role within a namespace.
- **edit**
  - Allows read/write access to most resources.
- **view**
  - Allows read-only access.

## How to Check the Status of the API Server?

```
kubectl get pods -n kube-system | grep kube-apiserver

kubectl describe pod kube-apiserver-controlplane -n kube-system

# For crictl (used by containerd and CRI-O)
sudo crictl ps -a | grep kube-apiserver
```

## How to configure Kubernetes Authorization method?

- To update the **--authorization-mode** flag of the kube-apiserver, you need to modify its configuration file, typically a static Pod manifest located at **/etc/kubernetes/manifests/kube-apiserver.yaml** on the control plane node.

### Step-XX: Connect to the Kubernetes control plane node(s) with administrative privileges over SSH

### Step-XX: Back up the configuration file

- Before making any changes, create a backup of the existing manifest file:

```
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml.backup
```

### Edit the kube-apiserver manifest

- Open the **/etc/kubernetes/manifests/kube-apiserver.yaml** file in a text editor (nano/vi):

```
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

```

### Step-XX: Modify the `--authorization-mode` flag of kube-apiserver

- Locate the `command:` section within the YAML file.
- Find the line that starts with `- --authorization-mode=` and modify the value to your desired comma-separated list of authorization modes (e.g., Node,RBAC).
- The order matters
  - The API server stops at the first _allow_ or _deny_ decision.

### Step-XX: Monitor the changes

- The Kubelet on the host monitors this directory. 
- It will automatically detect the change, terminate the existing kube-apiserver Pod, and start a new one with the updated configuration.

```
# To monitor the Pod cycle
watch crictl ps -a


# If using docker as a container runtime
watch docker ps -a

```
### Verify the new `kube-apiserver` configuration

- After the new Pod is running, verify that the *kube-apiserver* process is using the new authorization mode value:

```
# Find the process ID of the new kube-apiserver
ps aux | grep kube-apiserver | grep authorization-mode


# Alternatively, you can check the logs of the new pod
kubectl get pods -n kube-system -l component=kube-apiserver
kubectl logs <new-apiserver-pod-name> -n kube-system
```

## Hands-on Lab: Onboard a new User using TLS Certificates-based authentication and Authorization using RBAC

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

# Kubernetes `clusterRole` and `clusterRoleBinding`

In this section, you will learn how role based access control works. It covers the following concepts:

- Creating a _clusterRole_ with resource permissions
- Creating a _clusterRoleBinding_ object to associate a User with a Role

## Hands-on Lab: Create a `clusterRole` for a user and associate it to a User using `clusterRoleBinding`

### Step-01: Create a `clusterRole` for a User (cluster-admin)

- Develop a manifest, say _cluster-admin-clusterrole.yml_ for creating a clusterRole for an Admin user, such that she can:
  - View Nodes
  - Create Nodes
  - Delete Nodes
  - Update Nodes
  - List Nodes

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-role
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","get","create","update","delete"]
```

- Apply the manifest _cluster-admin-clusterrole.yml_

```
kubectl apply -f cluster-admin-clusterrole.yml

# List all the cluster roles
kubectl get clusterroles

# Describe the role to get more details
kubectl describe clusterrole cluster-admin-role
```

### Step-02: Create a `clusterRoleBinding` (for binding a user with a clusterRole)

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-rolebinding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-role
  apiGroup: rbac.authorization.k8s.io
```

- Apply the manifest _cluster-admin-clusterrolebinding.yml_

```
kubectl apply -f cluster-admin-clusterrolebinding.yml

# List all the clusterrolebindings
kubectl get clusterrolebindings

# Describe the rolebinding object to get more details
kubectl describe clusterrolebinding cluster-admin-rolebinding
```

### Verifying the access of cluster-admin User

```
# Check if the cluster-adin user can list nodes
kubectl auth can-i get nodes

# Check if the cluster-adin user can describe a node
kubectl auth can-i describe node
```

# Kubernetes Role Based Access Control (RBAC)

In this section, you will learn how role based access control works. It covers the following concepts:

- Creating a Role with resource permissions
- Creating a roleBinding object to associate a User with a Role

## Kubernetes RBAC - Key Concepts

- **Roles**
  - Role is a set of permissions within a specific namespace.
  - Ex. A _developer_ role in the default namespace can permit users to create and list pods, but not delete them.

- **ClusterRoles**
  - A ClusterRole is similar to a Role but is cluster-scoped.
  - It can grant permissions to cluster-wide resources (like nodes or persistent volumes) or to namespaced resources across all namespaces.

- **Subjects**
  - These are the entities that are granted permissions.
  - They can be:
    - User accounts
    - Groups of users (Group), or
    - ServiceAccounts (used by processes running in pods)
- **RoleBindings**
- **ClusterRoleBindings**

## Hands-on Lab: Create a `Role` for a Developer user and associate it to a User using `roleBinding`

### Create a Role object

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","get","create","update","delete"]
- apiGroups: [""]
  resources: ["configMap"]
  verbs: ["create","get","list"]
```

- Apply the above role manifest, say _developer-role.yml_

```
kubectl apply -f developer-role.yml

# List all the role from current namespace (default)
kubectl get roles

# Describe the role to get more details
kubectl describe role developer-role
```

### Create a roleBinding object

```
apiVersion: rbac.authorization.k8s.io/v1
kind: roleBinding
metadata:
  name: dev1-rolebinding
subjects:
- kind: User
  name: dev1
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

- Apply the above roleBinding manifest, say _dev1-roleBinding.yml_

```
kubectl apply -f dev1-roleBinding.yml

# List all the role from current namespace (default)
kubectl get rolebindings

# Describe the rolebinding to get more details
kubectl describe rolebinding dev1-rolebinding
```

### Verifying the access of a User (e.g developer)

```
# Check if the current user can create a Deployment object
kubectl auth can-i create deployment

# Check if the current user can delete a Pod object
kubectl auth can-i delete pod

# Check if a particular user can perform an action
kubectl auth can-i delete pod --as dev1

# Check if a particular user can perform an action in a particular namespace
kubectl auth can-i delete pod --as dev1 --namespace development
```

### How to allow a user/s access to a particular k8s resource only (e.g. pod, cm)?

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","get","create","update","delete"]
  resourceNames: ["dbpod"]          # Resource name to allow access to
- apiGroups: [""]
  resources: ["configMap"]
  verbs: ["create","get","list"]
  resourceNames: ["dbconfig","appconfig"]    # Resource name to allow access to
```

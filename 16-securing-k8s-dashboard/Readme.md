# Securing Kubernetes Dashboard

- `Important`
  - The official Kubernetes Dashboard project is archived and no longer actively maintained due to a lack of contributors. It has been moved to a "retired" repository.
  - **Reference**: https://github.com/kubernetes-retired/dashboard/blob/master/README.md
  - **Status**: Archived/Retired (as of 30 Jan 2026)
  - **Replacement**: Headlamp (https://headlamp.dev/) is the recommended open-source replacement

### Step 1: Deploy the Kubernetes Dashboard

- **Using K8s manifests**

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

```

- **Using Helm**

```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard

helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

### Step 2: Create a Service Account for K8s Dashboard Pod (with administrative access)

- create a new `Service Account` with the name _admin-user_ in namespace kubernetes-dashboard

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

- Create a `ClusterRoleBinding`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

- `Create a Bearer Token` for the above created ServiceAccount

```
kubectl -n kubernetes-dashboard create token admin-user

[The preceding command should print something like this: aW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2]
```

- We can also create a token with the secret which bound the service account and the token will be saved in the Secret

```
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"
type: kubernetes.io/service-account-token
```

### Step 3: Retrieve the Access Token

- After Secret is created, you may execute the following command to get the token which is saved in the Secret:

```
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath="{.data.token}" | base64 -d
```

### Step 4: Access the Dashboard UI using kubectl proxy

```
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard 8443:443
```

### Step 5: Log in

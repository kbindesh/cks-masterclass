# Service Accounts

## 01. Service Account - Overview

- A _ServiceAccount_ is a non-human identity used by Pods, applications, or services to authenticate and access resources, rather than a person.

- SA enable automated, secure, system-to-system interactions without interactive login.
- SA are crucial for cloud APIs, CI/CD pipelines, and background tasks.

## 02. Service Account - Key Concepts

- By default, k8s uses serviceAccounts to enable comminication between various k8s components.

- List all the existing serviceAccounts:

```
kubectl get sa -all-namespaces
```

- All the namespaces including built-in and custom namespaces comes with a _default_ serviceAccount.

### `default` ServiceAccount

- Every Namespace automatically gets a _default_ serviceAccount.

- **Automatic Assignment**
  - Whenever you create a new Pod in any _Namespace_, the _default_ _serviceAccount_ gets associated with it and inherit all the SA permissions.

- **API discovery permissions**
  - The _default_ serviceAccount in a Namespace has no permissions by default, other than the basic API discovery permissions granted to all authenticated users and follows the principle of least privilege.

- **No Inherent permissions**
  - By default, the _default_ serviceAccount cannot view, list, or modify other resources within the cluster, even within the same namespace.
  - If you attempts to do so will result in a "Forbidden" error (HTTP 403) unless explicit permissions have been granted via Role-Based Access Control (RBAC).

- **No Default _Token_ mount**
  - In Kubernetes versions v1.24 and above, a non-expiring secret token is no longer automatically created and mounted for the default service account, further enhancing security.

### Disable serviceAccount token auto-mounting at Pod level

### Disable serviceAccount token auto-mounting at serviceAccount level

## 03. Best Practices for serviceAccounts

- Avoid using the _default_ serviceAccount for apps that require access to the K8s API.

- Create dedicated, custom serviceAccounts for each app/pod or workload.

- Grant only the minimum necessary permissions (principle of least privilege) to the custom serviceAccounts using RBAC _Roles_ and _RoleBindings_.

- Disable automatic token mounting for _default_ serviceAccounts if possible, by setting `automountServiceAccountToken: false` on the serviceAccount definition.

## 04. Lab: Create a Pod in default Namespace and verify the _default_ serviceAccount & its permissions

```
# Create a Pod in default Namespace
kubectl run --image=nginx nginx-pod

kubectl get pods

kubectl describe pod nginx-pod
[observe the Service Account field (default)]
```

### 4.1 Now, get inside the pod and checkout the serviceAccount token

```
kubectl exec -it nginx-pod -- bash

cd /var/run/secrets/kubernetes.io/serviceaccount/

[You will find a token file]

# Save the token in a variable and then use it to explicitly make a callout to API Server (here it is https://10.0.0.10:6443)

cat token
token=$(cat token)

# Callout with a valid token
curl -k -H "Authorization: Bearer $token" https://10.0.0.10:6443/api/v1 => PASS

# Callout with an invalid token
curl -k -H "Authorization: Bearer sdf334" https://10.0.0.10:6443/api/v1 => FAIL

# List Namespaces | Not authorized
curl -k -H "Authorization: Bearer $token" https://10.0.0.10:6443/api/v1/namespaces => FAIL
```

## 05. Lab: Create a Pod and serviceAccount in a custom Namespace and elevate the Pod Access using RBAC

```
kubectl apply -f manifests

# List Pods, ServiceAccounts, Role and Role bindings of dev-ns Namespace
kubectl get po,sa,role,rolebindings -n dev-ns

# Get inside the created pod and try making a callout to access k8s apis
kubectl exec -n dev-ns -it pod-sa -- /bin/sh

# List Pods - PASS
kubectl get pods -n dev-ns

# List Namespaces - FAIL
kubectl get ns

# List Services - FAIL
kubectl get svc

```

## 06. Lab: Generate a Token for serviceAccount and share it with the Pod to make API callouts

- For Kubernetes versions 1.24 and later, tokens are bound to specific objects and have a limited lifespan by default.

### 6.1 Generate a time-bound token (recommended)

```
kubectl create token <service-account-name> -n <namespace>

# Generate a time-bound token for a ServiceAccount which will last for 2hrs
kubectl create token <service-account-name> -n <namespace> --duration=2h
```

### 6.2 Generate a long-lived (non-expiring) token

- This is not recommended due to security risks, but it is possible by manually creating a Secret object of type `kubernetes.io/service-account-token` with an annotation referencing the service account.

- Create a new Secret object of type _service account token_ using following manifest, secret-for-sa-token.yaml:

```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: api-service-account-token
  namespace: dev-ns
  annotations:
    kubernetes.io/service-account.name: webapp-sa
```

- Apply the above yaml manifest to created a Secret with token:

```
kubectl apply -f secret-for-sa-token.yaml

kubectl get secrets -n dev-ns

kubectl describe secret api-service-account-token -n dev-ns
```

- Retrieve the token (long-lived) from the above created Secret:

```
kubectl get secret api-service-account-token -n dev-ns -o=jsonpath='{.data.token}' | base64 --decode
```

## References

- https://kubernetes.io/docs/concepts/security/service-accounts/

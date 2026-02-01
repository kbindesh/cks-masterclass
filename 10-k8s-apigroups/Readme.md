# Kubernetes API Groups

## 01. Overview

- K8s _API groups_ organize related API resources into logical categories to make the API extensible and easier to manage and version.
- There are two primary types of API groups:
  1. **Core API Group**
  2. **Named API Groups**

### 1.1 `Core API Group` (also known as "legacy" or "empty string" group)

- This group contains the fundamental, essential building blocks of Kubernetes.
- **API Version** field: _apiVersion: v1_ (the group name is an empty string, so no prefix is used).
- **API Path**: /api/v1
- **Example Resources**
  - Pods
  - Services
  - Namespaces
  - Nodes
  - ConfigMaps
  - Secrets
  - PersistentVolumes

### 1.2 `Named API Groups`

- These groups are used for modular, newer, or extended features.
- They have a specific name (e.g., apps, batch) to avoid conflicts with the core group and each other.
- **API Version** field: _apiVersion: $GROUP_NAME/$VERSION_ (e.g., apiVersion: apps/v1)
- API Path: /apis/$GROUP_NAME/$VERSION (e.g., /apis/apps/v1).
- Examples of Named Groups and Resources
  - **apps**: Deployments, StatefulSets, DaemonSets, ReplicaSets
  - **batch**: Jobs, CronJobs
  - **networking.k8s.io**: Ingresses, NetworkPolicies
  - **rbac.authorization.k8s.io**: Roles, ClusterRoles, RoleBindings, ClusterRoleBindings
  - **storage.k8s.io**: StorageClasses, VolumeAttachments
  - **autoscaling**: HorizontalPodAutoscalers

## 02. Viewing `API Groups` & `API Resources`

```
# List all the K8s API Resources, associated groups & verbs
kubectl api-resources -o wide

# List API Groups
sudo curl https://10.0.0.10:6443 -k --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key --cacert /etc/kubernetes/pki/ca.crt

# List all the API Resources with core API v1
sudo curl https://10.0.0.10:6443/api/v1 -k --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key --cacert /etc/kubernetes/pki/ca.crt

# List Pods
sudo curl https://10.0.0.10:6443/api/v1/pods --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key --cacert /etc/kubernetes/pki/ca.crt

# List Deployments
sudo curl https://10.0.0.10:6443/apis/apps/v1/deployments --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key --cacert /etc/kubernetes/pki/ca.crt
```

## 03. API Groups

<table>
    <tr>
        <th>apiGroup</th>
        <th>Version</th>
        <th>API Resources</th>
    </tr>
    <tr>
        <td>(empty string) “”</td>
        <td>v1 [ This is the default apiGroup, so we keep it blank]</td>
        <td>pods, services, configmaps, secrets, namespaces, persistentvolumeclaims, endpoints</td>
    </tr>
    <tr>
        <td>apps</td>
        <td>v1</td>
        <td>deployments, replicasets, statefulsets, daemonsets</td>
    </tr>
    <tr>
        <td>batch</td>
        <td>v1, v1beta1</td>
        <td>jobs, cronjobs</td>
    </tr>
    <tr>
        <td>autoscaling</td>
        <td>v1</td>
        <td>horizontalpodautoscalers</td>
    </tr>
    <tr>
        <td>networking.k8s.io</td>
        <td>v1</td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td>ingresses, networkpolicies</td>
    </tr>
    <tr>
        <td>policy</td>
        <td>v1</td>
        <td>poddisruptionbudgets</td>
    </tr>
    <tr>
        <td>rbac.authorization.k8s.io</td>
        <td>v1</td>
        <td>roles, clusterroles, rolebindings, clusterrolebindings</td>
    </tr>
    <tr>
        <td>scheduling.k8s.io</td>
        <td>v1</td>
        <td>priorityclasses</td>
    </tr>
    <tr>
        <td>storage.k8s.io</td>
        <td>v1</td>
        <td>storageclasses, volumeattachments, csidrivers, csinodes</td>
    </tr>
    <tr>
        <td>admissionregistration.k8s.io</td>
        <td>v1</td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td>mutatingwebhookconfigurations, validatingwebhookconfigurations</td>
    </tr>
    <tr>
        <td>apiextensions.k8s.io</td>
        <td>v1</td>
        <td>customresourcedefinitions</td>
    </tr>
    <tr>
        <td>certificates.k8s.io</td>
        <td>v1</td>
        <td>certificatesigningrequests</td>
    </tr>
</table>

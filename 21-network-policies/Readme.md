# Network Policies

- _NetworkPolicies_ allow you to specify rules for traffic flow within your cluster, and also between Pods and the outside world.
- Kubernetes does not enforce network policies.
- A Container Network Interface (CNI) plugin that supports network policies (e.g., Calico, Cilium, Weave Net) must be installed and running in the cluster for the policies to work.
- Network policies operate at the IP address and port level (OSI layers 3 and 4), managing TCP, UDP, and SCTP traffic

## Key components of Network Policy Object

- A _NetworkPolicy_ object uses standard YAML structure and has several key fields in its _spec_ section, namely:
  - **podSelector**
    - Specifies which pods the policy applies to, using label selectors.
    - An empty selector ({}) matches all pods within the namespace.

  - **policyTypes**
    - Include _Ingress_, _Egress_, or both the values, indicating which types of traffic the policy governs.
    - As a recommended practice, you must explicitly mention the _policyType_.

  - **ingress**
    - A list of rules that define allowed incoming traffic to the selected pods.
    - Traffic sources can be defined by:
      - **ipBlock** (CIDR ranges)
      - **podSelector** (pods in the same namespace)
      - **namespaceSelector** (pods in other namespaces)
      - **ports** (specific protocols and port numbers)
  - **egress**
    - A list of rules that define allowed outgoing traffic from the selected pods
    - Traffic destinations can be defined by:
      - **ipBlock** (CIDR ranges)
      - **podSelector** (pods in the same namespace)
      - **namespaceSelector** (pods in other namespaces)
      - **ports** (specific protocols and port numbers)

## `CNI Plugins` that supports Kubernetes `Network Policies`

- Reference
  - https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/

- Popular CNI plugins those are compatible with Kubernetes Network Policies include:
  - **[Calico](https://www.tigera.io/project-calico/)**: A widely used CNI that offers robust security policy enforcement. It also provides its own custom resource definitions (CRDs) for more advanced policies beyond the standard Kubernetes API.

  - **[Cilium](https://cilium.io/)**: A modern, high-performance solution that leverages eBPF for networking, observability, and security, including advanced L3-L7 network policies.

  - **Weave Net**: A simple and reliable option that supports network policies.

  - **Amazon VPC CNI**: The AWS-specific CNI now supports Kubernetes Network Policies in recent versions (v1.14.0 and later) when the network policy controller is enabled.

## Hands-on: Create Network Policies for restricting access for Pods

### Prerequisites

- **Compatible Container Network Interface (CNI) plugin**
  - To use network policies, you must have a networking solution which supports NetworkPolicy.
  - For supported CNI list, you may refer https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
- Kubernetes Cluster
- kubectl

### Deploy Applications pods with Labels

- This application deployment will create the following objects:
  - frontend app _Pod_ (nginx)
  - clusterIp _Service_ for frontend app
  - backend app _Pod_ (nginx)
  - clusterIp _Service_ for backend app
  - database _Pod_ (mysql)
  - clusterIp _Service_ for database app

```
kubectl apply -f manifests/3-tier-app-deploy.yml

# List all the Pods and Services
kubectl get po,svc
```

### Test Pod-to-Pod connectivity

```
# Check the connectivity from frontend pod to backend pod
kubectl exec -it frontend -- curl backend:80

# Check the connectivity from frontend to database Pod

kubectl exec -it frontend -- telnet db 3306

[The preceding command will allow frontend pod to connect to db pod]

# (optional) To install Telnet
kubectl exec -it frontend bash
apt-get update && apt-get install telnet

```

### Create a `Network Policy` for `Database` App and Test the connectivity

```
kubectl apply -f manifests/db-network-policy.yml

# List all the network policies
kubectl get netpol

kubectl describe netpol db-network-policy
```

- Test the connectivity from **frontend** Pod to **database** Pod

```
# Check the connectivity from frontend to database Pod

kubectl exec -it frontend -- telnet db 3306

[This time as the Network Policy is in place, communication is not going to happen]
```

- Test the connectivity from **backend** Pod to **database** Pod

```
kubectl exec -it frontend -- telnet db 3306

[As the Network Policy allow backend pod to communicate with data base pod, communication is going to happen]
```

### Create a Network Policy with custom ingress and egress rules and Test the connectivity

### Configure `Network Policy` for selective Pod communication

### Create a `Network Policy` with rules to control traffic based on `CIDR ranges`

For sample manifests, kindly refer [network-policy-with-ipblock.yml](manifests/network-policy-with-ipblock.yml)

### Create a Network Policy with a `namespaceSelector` to control traffic between pods in different namespaces

## Network Policy - Best Practices

- **Implement a `default-deny` Policy**
  - A common and highly recommended security practice is to start with a policy that denies all ingress and egress traffic for a namespace, then incrementally add explicit "allow" rules for necessary communication.
- Restrict communication between different application tiers.
  - e.g., a frontend pod can only talk to its designated backend pods and not directly to a database pod))
- Control cross-Namespace Communication
  - Use **namespaceSelector** to limit which namespaces can send or receive traffic from a given set of pods.
- **Use Labels carefully**
  - Network policies rely heavily on object _labels_ for selecting pods and namespaces.
  - Well-planned labeling is key to managing policies efficiently.

## Troubleshooting Network Policies

6. Troubleshooting Network Policies

- **Check Network Plugin**
  - Ensure your CNI plugin supports Network Policies.
- **Pod Labels**
  - Verify that pod _labels_ match the _selectors_ in your Network Policy.
- **Policy Types**
  - Ensure you've specified the correct policy types (Ingress, Egress).
- **Logs and Events**
  - Check the logs and events for errors related to Network Policies.

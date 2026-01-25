# Setup Kubernetes cluster using `kubeadm` (for CKA & CKS certification exams)

## Kubernetes Cluster Setup

- Master (atleast one)
  - should have a minimum of 2 vCPU and 2GB RAM

- Worker (atleast one | You can add more worker nodes)
  - should have a minimum of 1 vCPU and 2GB RAM is recommended

- 10.X.X.X/X network range IPs for master and worker nodes.

- We will use the 192.x.x.x series as the pod network range that will be used by the Calico network plugin.
  - Make sure the Node IP range and pod IP range do not overlap.

## kubeadm - Nodes Port Specifications

- Reference: https://kubernetes.io/docs/reference/networking/ports-and-protocols/

![control-plane-portspecs](images\ctrlplnportspec.png)
</br>
![worker-node-portspec](images\workerportspec.png)

## Setting-up the K8s cluster

- Enable iptables Bridged Traffic on all the Nodes
- Disable swap on all the Nodes
- Install container runtime on all nodes- We will be using containerd
- Install Kubeadm, Kubelet on all the nodes.
- Install kubectl on you client machine and control plane node
- Initiate Kubeadm control plane configuration on the master node
- Save the node join command with the token
- Install the Calico network plugin (operator).
- Join the worker node to the master node (control plane) using the join command
- Validate all cluster components and nodes.
- Install Kubernetes Metrics Server
- Deploy a sample app and validate the app

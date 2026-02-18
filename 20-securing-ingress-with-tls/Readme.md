# Securing `Ingress` with `TLS`

## 01. Why Ingress?

### K8s solutions to expose HTTP services using without Ingress

- K8s **Service** allows us to access Pods or set of Pods with an endpoint, within or from outside k8s cluster

- Services can be exposed to the outside world using following `Service` types:
  - **NodePort** (in port range 30000-32767)
  - **LoadBalancer** (with external load balancer)

### Issues with `NodePort` and `LoadBalancer` services for exposing HTTP services

- With **NodePort** service, end-user must specify the node port along with the endpoint.

- **LoadBalancer** service is good, however has some issues:
  - They are not available in all the env.
  - They usually works at OSI Layer-4 (IP+Port) and not at Layer-7 (HTTP/S)
  - Additional cost is involved
  - Requires additional step for DNS Integration

- **Reverse proxy** - HAProxy, Apache, NGINX, Traefik
  - Tedious to configure and manage

## 02. Kubernetes `Ingress` API Resource Overview

- _Ingress_ is designed to expose HTTP services

- **Ingress**
  - API Resource: Ingress
  - apiVersion: networking.k8s.io/v1
  - Kind: Ingress
  - Alias: ing
  - Namespace Scoped: TRUE

```
# Get the list of all the Ingress objects
kubectl get ingress
OR
kubectl get ing
```

### `Ingress` Key Features

- **Host-based Routing**
  - e.g. api.example.com goes to the api-service, and web.example.com goes to the web-service
- **Path-based Routing**
  - e.g. example.com/api goes to _api-service_, and example.com/app goes to _app-service_
- **SSL/TLS Termination**
  - Handling the decryption of HTTPS traffic at the Ingress layer, offloading the computational burden from backend application pods and centralizing certificate management.

- **Load Balancing**
  - Distributing incoming requests across multiple healthy pods of a service to ensure high availability and scalability.

- **Security and Access Control**

- **Rewrite and Redirect Rules**

### Ingress Key concepts

- **Ingress Controller**
  - This is an application (usually a reverse proxy and load balancer, such as NGINX, Traefik, or HAProxy) that runs as a pod within the cluster and enforces the rules defined in the Ingress resource.
  - It watches for Ingress objects via the Kubernetes API and configures its underlying proxy to route traffic accordingly.
- **Ingress Resource** (ing)
  - This is a Kubernetes API object where you define the rules for routing incoming external traffic.

- **Service**
  - The Ingress controller routes traffic to a stable Kubernetes Service (typically ClusterIP type), which then load balances the requests across the appropriate backend pods.

- **Ingress Backend**
  - A destination Service and port to which traffic should be forwarded when a rule match is found.

- **IngressClass**

### Supported `Ingress Controllers`

- To get the list of supported Ingress controllers, you may refer to: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

- Commonly used Ingress controllers are:
  - AWS Load Balancer Controller
  - NGINX Ingress Controller for Kubernetes
  - AKS Application Gateway Ingress Controller
  - Ingress GCE
  - Traefik

## Reference

- Official nginx-ingress GitHub Repository
  - https://github.com/kubernetes/ingress-nginx

- **nginx-ingress** Deployment manifests
  - https://github.com/kubernetes/ingress-nginx/tree/main/deploy/static/provider

- **nginx-ingress** Baremetal Deployment manifest
  - https://github.com/kubernetes/ingress-nginx/blob/main/deploy/static/provider/baremetal/deploy.yaml

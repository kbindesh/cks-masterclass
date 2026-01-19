# TLS in Kubernetes

### Certificate Authority (CA) Certificate for Kubernetes

- ca.crt
- ca.key

## Server Certificates for Kubernetes server components

- kubeapi-server
- etcd
- kubelet

## Client Certificates for Kubernetes client components

- users (e.g. admin)
- scheduler
- kube-controller-manager
- kube-proxy
- kube-api server to etcd
- kube-api server to kubelet
- kubelet to kube-api server

## Lab: Create Certificates in Kubernetes using `openssl`

### Create private key and certificate for Kubernetes `Certificate Authority (CA)`

```
# Step-01: Generate private key
openssl genrsa -out ca.key

# Step-02: Generate Certificate Signing Request (csr) - sertificate with no signature
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

# Step-03: Sign the above created Certificate (.csr) using X509 - self-signed
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

### Create private key and certificate for Kubernetes Certificate Authority (CA)

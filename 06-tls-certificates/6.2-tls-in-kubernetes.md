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

## Certificate expiry and management

- `Important Note`: kubeadm cannot manage certificates signed by an external CA.

```
# Use the check-expiration subcommand to check when certificates expire
kubeadm certs check-expiration
```

## Automatic certificate renewal

- kubeadm renews all the certificates during control plane upgrade
- For more complex requirements with respect to certificate renewal, you can opt out from the default behavior by passing **--certificate-renewal=false** to _kubeadm upgrade apply_ or to _kubeadm upgrade node_

## Manual certificate renewal

### Renew certificates with the Kubernetes certificates API

### Renew certificates with external CA

## References

- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

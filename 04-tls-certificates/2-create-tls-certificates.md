# Generating TLS Certificates for Kubernetes

- In this section, you will learn how to use _openssl_ utility to generate and sign certificates.

## 01. Generate TLS Certificates for K8s `Server` components

### Generate TLS Certificate for CA (self-signed)

```
# Generate private key for CA - ca.key
openssl genrsa -out ca.key 2048

# Generate Certificate signing request - ca.csr
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

# Sign above created certificate using above private key (self-signed) - ca.crt
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt

# Inspect/Describe the Certificate details | Observe the Common Name (CN) & Organization (O)
openssl x509 -in ca.crt -text -noout
```

### Generate TLS Certificate for Admin User (self-signed)

```
# Generate private key for admin user - admin.key
openssl genrsa -out admin.key 2048

# Generate Certificate signing request - admin.csr (without group spec)
openssl req -new -key admin.key -subj "/CN=cluster-admin" -out admin.csr

# (Optional) Generate Certificate signing request - admin.csr (with k8s group specs)
openssl req -new -key admin.key -subj "/CN=cluster-admin/O=system:masters" -out admin.csr

# Sign above created certificate using CA certificate & CA key  (self-signed) - admin.crt
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt

# Inspect/Describe the Certificate details | Observe the Common Name (CN) & Organization (O)
openssl x509 -in admin.crt -text -noout
```

### Generate TLS Certificate for kube-api-server

```

```

## 02. Generate TLS Certificates for K8s `Client` components

### kube-scheduler

### kube-controller-manager

### kube-proxy

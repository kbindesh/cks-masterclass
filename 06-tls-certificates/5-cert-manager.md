# Managing Application TLS/SSL Certificates using cert-manager

## Install Helm

```bash
sudo dnf install -y git

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4

chmod 700 get_helm.sh

./get_helm.sh
```

## Install `cert-manager`

```bash
# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm upgrade cert-manager jetstack/cert-manager `
             --install `
             --create-namespace `
             --namespace cert-manager `
             --set installCRDs=true `
             --set nodeSelector."kubernetes\.io/os"=linux


# Verify the Installation
kubectl get all -n cert-manager
```

## Create a `ClusterIssuer` and `Certificate`

- Create two new files, namely [clusterissuer-selfsigned.yaml](manifests/clusterIssuer-selfsigned.yaml) and [certificate.yaml](manifests/certificate.yaml) and deploy it:

```
kubectl apply -f clusterissuer-selfsigned.yaml

kubectl apply -f .\certificate.yaml

# Get the list of all the certificates, secrets and clusterIssuers
kubectl get certificate,secret,ClusterIssuer

[A new certificate, secret and Clusterissuer will be created]


# Describe secret to see the generated TLS certificates
kubectl describe secret app01-tls-cert-secret

[You should see 3 files, ca.crt, tls.crt, tls.key]
```

## Deploy and Application

- Create a new file [app-deploy-svc.yaml](manifests/app-deployment-and-svc.yaml) and deploy it on K8s cluster:

```bash
kubectl apply -f app-deploy-svc.yaml

# List objects to verify the deployment
kubectl get pod,svc
```

## Verify the TLS configuration

```bash
kubectl run nginx --image=nginx

kubectl exec -it nginx -- curl --insecure https://app01.default.svc.cluster.local
# Hello, world!
# Protocol: HTTP/2.0!
# Hostname: app01-7f265d2986-vzx2z

# Verify TLS certificate
kubectl exec -it nginx -- curl --insecure -v https://app01.default.svc.cluster.local
```

## Extras

```bash
# You can retrieve the Cluster Certificate Authority data using the AWS CLI
aws eks describe-cluster --name <your-cluster-name> --query "cluster.certificateAuthority.data" --output text


# Certificates on Worker Nodes
/etc/kubernetes/pki/: General PKI directory (on some builds, though most are managed).
/var/lib/kubelet/pki/: Contains the kubelet client and server certificates.
/etc/kubernetes/bootstrap-kubeconfig: Contains the initial bootstrap credentials for the node

# list active Certificate Signing Requests (CSRs) directly via kubectl
kubectl get csr

# Get Application-Level Certificates (Cert-Manager)
List all certificates: kubectl get certificate --all-namespaces
List certificate issuers: kubectl get issuers,clusterissuers --all-namespaces

# Secrets (Where TLS is stored)
kubectl get secrets --field-selector type=kubernetes.io/tls --all-namespaces
```

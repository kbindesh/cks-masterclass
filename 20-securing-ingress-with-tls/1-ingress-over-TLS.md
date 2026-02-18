## Configure NGINX Ingress on K8s (kubeadm) and Access Application over TLS

This section demonstrate the following concepts:

- Deploy NGINX Ingress controller
- Create Ingress object
- Enabling TLS for Ingress
- Deploying a sample application and expose it using nginx Ingress

### Step-01: Prerequisites

- Kubernetes cluster
- kubectl
- A domain name pointing to one or more of your cluster node's IP addresses

### Step-02: Deploy an NGINX Ingress Controller

- Reference
  - https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/baremetal.md
  - https://github.com/kubernetes/ingress-nginx/blob/main/deploy/static/provider/baremetal/deploy.yaml

```
# For kubeadm based cluster, deploy ingress controller with nodePort service
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/refs/heads/main/deploy/static/provider/baremetal/deploy.yaml
```

### Step-03: Validate the NGINX Ingress Controller objects

```
kubectl get all -n ingress-nginx

kubectl get svc -n ingress-nginx
```

### Step-04: Generate a Private Key and a TLS Certificate using OpenSSL

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ingress-tls.key -out ingress-tls.crt -subj "/CN=www.ingress-tls.com/O=tls-example-org" -addext "subjectAltName = DNS:www.ingress-tls.com"

[The preceding command will generate 2 files: a private key (.key) and a Certificate (.crt)]
```

`Alternativly`, you can also generate the _private key_ and a _self-signed certificate_ (not recommended for production) using online tool like https://regery.com/en/security/ssl-tools/self-signed-certificate-generator:

- **Site Name**: api.flaskapp.local
- Click on Generate button >> Download both the _.key_ and _.crt_ files

### Step-05: Create a new Secret for storing TLS Certificate and Private Key

- Create a new secret for storing private and TLS certificate to be used by Ingress

```
# Create a K8s Secret | type=kubernetes.io/tls
kubectl create secret tls api-flaskapp-local-cert --key api-flaskapp.local-
privateKey.key --cert api-flaskapp.local.crt -n flask-app

# Verify and Inspect above created secret
kubectl get secret api-flaskapp-local-cert -o yaml -n flask-app
```

### Step-06: Deploy the Applications and Services

- The deployment of application will create the following objects:
  - flask-app _Namespace_
  - blue app _Deployment_
  - blue app _clusterIP_ Service
  - yellow app _Deployment_
  - yellow app _clusterIP_ Service

- For all the relavant manifests, kindly refer [manifests](./manifests/) directory:

```
kubectl apply -f manifests/01-namespace.yml
kubectl apply -f manifests/02-blue-app-and-cip-svc.yml
kubectl apply -f manifests/03-yellow-app-and-cip-svc.yml

# List all the namespaces | Observe the "flask-app" Namespace in the list
kubectl get ns

# Verify the Application related K8s objects
kubectl get po,deploy,svc -n flask-app
```

### Step-07: Create an `Ingress` object with TLS Configuration

- Create ingress object for publishing Python Flask Apps:

```
kubectl apply -f manifests/04-flaskapp-ingress.yml

kubectl get ing -n flask-app

kubectl describe ing flask-app-ingress
```

### Step-08: Access the Application

- As the NGINX Ingress is configured with _nodePort_ Service, kindly use node port of Ingress controller to access the application.

```
# Get the list of all the services from ingress-nginx namespace
kubectl get svc -n ingress-nginx

[Make a note of node ports (for both 80 & 443) of ingress-nginx nodePort Service]

# Get the list of all the K8s Nodes with IP Address
kubectl get nodes -o wide

[Make a note of K8s Node's IP Address]
```

- Now, as we have all the neccessary informationm let's access the application:

#### Open browser or any REST client and hit the following URL to access `Blue Application`:

```
# Syntax - For accessing Blue App (over HTTP)
http://{WORKER_NODE_IP}:{INGRESS_NODE_PORT}/api/blue

# Example-01 - Acessing Blue App over insecure HTTP connection
http://10.0.0.11:32507/api/blue

# Example-02 - Accessing Blue App over secure HTTPS connection
https://10.0.0.11:32372/api/blue
```

#### Open browser or any REST client and hit the following URL to access `Yellow Application`:

```
# Syntax - For accessing Yellow App (over HTTP)
http://{WORKER_NODE_IP}:{INGRESS_NODE_PORT}/api/yellow

# Example-01 - Acessing Yellow App over insecure HTTP connection
http://10.0.0.11:32507/api/yellow

# Example-02 - Accessing Yellow App over secure HTTPS connection
https://10.0.0.11:32372/api/yellow

```

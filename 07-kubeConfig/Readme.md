# Managing access to K8s cluster using `kubeconfig` file

- A kubeconfig file is a YAML-formatted configuration file used by kubectl and other client apps to manage cluster access, authentication, and contexts.

- By default, kubectl looks for kube configuration at following location:
  - Linux/MacOS: ~/.kube/config.
  - Windows: %USERPROFILE%\.kube\config

## 01. `kubeconfig` file structure

The kubeconfig file has the following sections:

- **apiVersion**
  - Specifies the version of the config (usually v1).
- **kind**
  - The object type, which is always Config.
- **clusters**
  - A list of Kubernetes API server endpoints and their CA certificates.
- **users**
  - A list of authentication credentials (tokens, certs, or external auth).
- **contexts**
  - A list of named profiles that map a specific User to a specific Cluster.
- **current-context**
  - The active profile that kubectl uses by default.
- **preferences**
  - General CLI settings (e.g., color settings).

## 02. kubeconfig file - Key concepts

### 2.1: clusters

- Cluster section tells the "Where" (which k8s cluster to connect to).
- It contains the connection details for one or more Kubernetes clusters.
  - **server**: The URL of the Kubernetes API server (e.g., https://XX.XX.XX.XX:6443).
  - **certificate-authority**: Path to the CA cert file used to verify the server.
  - **certificate-authority-data**: The base64-encoded content of the CA certificate.
  - **insecure-skip-tls-verify**: A boolean to bypass TLS verification (not recommended for production).

### 2.2: users

- Users section defines the "Who".
- It stores the credentials for authenticating against the cluster.
  - **client-certificate / client-key**: Paths to client-side TLS certificates.
  - **token**: A static bearer token for authentication.
  - **exec**: (Recommended for cloud) A command that calls an external provider (like AWS, GKE, or Azure) to fetch a temporary token.
  - **auth-provider**: Specific configurations for cloud-managed identity providers.

### 2.3: contexts

- Contexts section defines the "How".
- It allows you to create specific configuration by grouping a cluster, user, and optionally a default namespace.
  - **cluster**: References a name from the clusters list.
  - **user**: References a name from the users list.
  - **namespace**: Sets the default namespace for commands run in this context.

## 03. kubeconfig - Essential Commands

```
# View current config
kubectl config view

# List all the kube config contexts
kubectl config get-contexts

# Check active context
kubectl config current-context

# Switch to any kubeconfig context
kubectl config use-context <context-name>

# Add the new TLS-based user credentials to the kubeconfig file
kubectl config set-credentials $USER_NAME --client-certificate=$USER_NAME.crt --client-key=$USER_NAME.key --embed-certs=true

# Create a new context
kubectl config set-context $CONTEXT_LABEL  --cluster=kubernetes --user=$USER_NAME --namespace=default
```

## 04. Sample kubeconfig file

```

```

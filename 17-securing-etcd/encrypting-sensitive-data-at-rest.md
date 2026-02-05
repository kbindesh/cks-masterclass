# Encrypting sensitive information at rest in Kubernetes

## Hands-on Lab: Configure Kubernetes for Encrypting `Secrets` at rest

### Step-01: Prerequisites

- Kubernetes cluster with admin privileges
- etcdctl
- kubectl
- openssl or urandom

### Step-02: Create a sample file to be stored in Secret

```
# Create a sample file to be stored in secret
$ echo "This is my secret content" > example
```

### Step-03: Create a Secret

```
# Create a Secret from above file
$ kubectl create secret generic my-secret --from-file example --from-literal=passphrase=topsecret

[Output: secret/my-secret created]

# Verify the Secret
$ kubectl get secret my-secret -o yaml
OR
$ kubectl describe secret my-secret
[The output will show the Base64-encoded value under the key example]
```

### Step-04: Query Secret data from etcd (unecrypted)

- `Important`: By default, the etcd encryption at rest is _disabled_, hence secret values remain only base64-encoded, which can allow anyone with access to etcd to decode them easily.

```
$ etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret | hexdump -C


[Specify the secret's etcd key path: /registry/secrets/<namespace>/<secret-name>]
[hexdump: for displaying the raw data in hexadecimal format. You should be able to read the secrets data]
```

### Step-05: Enable Encryption at Rest

```
# Check --encryption-provider-config property already set on kube-apiserver
$ kubectl describe po kube-apiserver-controlplane -n kube-system | grep "encryption-provider-config"

[If nothing returns, it means encryption at rest is diabled]
```

- **Generate a random 32-byte key (base64-encoded)**:

```
$ openssl rand -base64 32
OR
$ head -c 32 /dev/urandom | base64

[Save the key hash at safe place | we'll need it in the next step]
```

- Now, create a YAML file on your control plane node **/etc/kubernetes/enc/enc.yaml** with the following content (replace the sample key with your key generated in the previous step):

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: pC2AaBY/vBpjkQHQXUChP+OgYZ5m3eLjeTrMcCKyQsM=  # Replace with your generated key
      - identity: {}
```

### Step-06: Mount the encryption configuration file on kube-apiserver

- Since kubeadm runs the API server as a static pod, you must edit its manifest at **/etc/kubernetes/manifests/kube-apiserver.yaml**

- **Add an Argument**
  - Add **--encryption-provider-config=/etc/kubernetes/enc/enc.yaml** to the command section.

- **Add a Volume Mount**

```
volumeMounts:
- name: enc-config
  mountPath: /etc/kubernetes/enc
  readOnly: true
```

- **Add Volume**
  - Define the host path

```
volumes:
- name: enc-config
  hostPath:
    path: /etc/kubernetes/enc
    type: DirectoryOrCreate
```

`INFO`: The kube-apiserver will automatically restart once you save the changes

- After updating an argument, volume and volumeMount in /etc/kubernetes/manifests/kube-apiserver.yaml, the manifest spec section would look something as below:

```
spec:
  containers:
  - command:
    - kube-apiserver
    # ... other arguments ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
    volumeMounts:
      - name: enc
        mountPath: /etc/kubernetes/enc
        readOnly: true
  volumes:
    - name: enc
      hostPath:
        path: /etc/kubernetes/enc
        type: DirectoryOrCreate
```

- I've also attached a sample [kube-apiserver.yaml](./kube-apiserver.yaml) file.

- Save the changes to the **/etc/kubernetes/manifests/kube-apiserver.yaml** file. The kubelet, which manages static pods, will automatically detect the changes and recreate the _kube-apiserver-controlplane_ pod.

- In order to verify the changes you made, you can check the status of _kube-apiserver-controlplane_ pod or the associated container.

```
# Check the status of kube-apiserver-controlplane Pod | should be "Running"
kubectl get pods -n kube-system

# Check the status of kube-apiserver-controlplane Container | should be "Running"
sudo crictl ps -a | grep kube-apiserver
```

### Step-07: Verifying Encryption

- Once encryption is activated, any new secret you create will be encrypted at rest.

- Let's create a new secret to verify.

```
kubectl create secret generic encryptedsecret --from-literal=pwd=supersecretpassword

# Get the list of secrets
kubectl get secret

# Now, fetch the "pwd" key from etcd to ensure the secret's value is encrypted
$ etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/encryptedsecret | hexdump -C

[You will observe that the secret's data is no longer appears to be in the plain text and prefixed with k8s:enc:aescbc:v1:]
```

![encrypted-secret](images/encrypted-secret.png)

- In order to encrypt old/existing Secrets, run the following command:

```
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

## 02. FAQs

### 01. What are the various providers used in k8s EncryptionConfiguration?

- The **EncryptionConfiguration** uses a list of providers to specify how data is encrypted in etcd.
- The order of the providers in the list is crucial:
  - the first one in the list is used for encryption of new data, while all providers are tried in order for decryption.

- The available providers are:
  - **identity**
    - This is the default provider if no configuration file is specified. It provides no encryption, storing resources as-is in plain text.

  - **aescbc**
    - Uses AES-CBC (Cipher Block Chaining) with PKCS#7 padding. This method is fast but considered weak due to vulnerabilities to padding oracle attacks and is not recommended.

  - **aesgcm**
    - Uses AES-GCM (Galois/Counter Mode) with a random nonce. It is faster than aescbc but requires key rotation every 200,000 writes.
  - **secretbox**
    - Implements encryption using XSalsa20 and Poly1305. It offers strong security but uses relatively new technology that might not be acceptable in highly regulated environments.

  - **kms (v1 and v2)**
    - These providers use an envelope encryption scheme by integrating with an external Key Management Service (KMS), such as AWS KMS, Azure Key Vault, or Google Cloud KMS.
    - K8s uses a data encryption key (DEK) to encrypt the data, and the DEK is itself encrypted by a key encryption key (KEK) managed by the external KMS.

### 02. What are the various resources supported in k8s EncryptionConfiguration?

- The _EncryptionConfiguration_ object is used to define which resources stored in etcd are encrypted at rest.
- Though it is primarily used for **Secrets**, it can be used to encrypt other resources.

- Supported Resources and Formats
  - **secrets (Core Group)**
    - The most common use case, intended for protecting sensitive credentials.
  - **configmaps (Core Group)**
    - Can be encrypted to protect non-sensitive but confidential configuration data.
  - **Custom Resources (CRDs)**
    - You can specify custom resources for encryption, provided your cluster is running Kubernetes v1.26 or newer.
  - **Wildcards**
    - You can use wildcard syntax to define scope:

    ```
    *.* (or *): Encrypts all resources in all groups, including built-in and custom resources.

    *.<group>: Encrypts all resources within a specific group (e.g., *apps).

    * (in the core group): Used to encrypt all resources in the core group
    ```

## References

- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

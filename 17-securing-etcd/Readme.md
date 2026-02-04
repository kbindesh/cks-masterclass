# etcd Security Guidelines

## Overview

- Default port on etcd service listens: 2379

### Key Files and Locations

- **ca certificate** (--cacert) - /etc/kubernetes/pki/etcd/ca.crt
- **server certificate** - /etc/kubernetes/pki/etcd/server.crt
- **server key** - /etc/kubernetes/pki/etcd/server.key
- **etcd pod manifest** - /etc/kubernetes/manifests/etcd.yaml
  - In kubeadm based cluster, _etcd_ often runs as a static pod, and its configuration is defined within this manifest file, which points to other configuration files and certificates.

### Check the etcd service status

- Let's check if **etcd** service is running or not:

```
netstat -ntlp
OR
sudo ss -lptn 'sport = :2379'
OR
sudo lsof -i :2379
```

### Install etcdctl

- In order to get/put data into etcd, you need _etcdctl_ utility.
- In order to install etcdctl, log in to the control plane.
- Install Required Tools - **etcdctl** and **etcdutl** to backup and restore ETCD and execute the following script:

```
ARCH=$(uname -m)
if [ "$ARCH" = "x86_64" ]; then
    ARCH_TYPE="amd64"
elif [ "$ARCH" = "aarch64" ]; then
    ARCH_TYPE="arm64"
else
    echo "Unsupported architecture: $ARCH"
    exit 1
fi

ETCD_VER=v3.6.7
DOWNLOAD_URL=https://storage.googleapis.com/etcd
DOWNLOAD_FILE=etcd-${ETCD_VER}-linux-${ARCH_TYPE}.tar.gz

mkdir -p /tmp/etcd-download
curl -L ${DOWNLOAD_URL}/${ETCD_VER}/${DOWNLOAD_FILE} -o /tmp/${DOWNLOAD_FILE}

tar xzvf /tmp/${DOWNLOAD_FILE} -C /tmp/etcd-download --strip-components=1

sudo mv /tmp/etcd-download/etcd /tmp/etcd-download/etcdctl /tmp/etcd-download/etcdutl /usr/local/bin/
rm -f /tmp/${DOWNLOAD_FILE}
rm -rf /tmp/etcd-download

```

- **Verify the etcdctl Installation**

```
etcdctl version

etcdutl version
```

### Get required information for accessing etcd

- We need at least following four bits of information to securely access etcdctl:
  - **etcd endpoint** (--endpoints)
  - **ca certificate** (--cacert)
  - **server certificate** (--cert)
  - **server key** (--key)

- You can get the above details by describing the etcd pod running in the kube-system namespace:

```
# List all the pods from kube-system NS
kubectl get po -n kube-system

# Describe etcd pod and get the required details
kubectl describe pod etcd-controlplane -n kube-system

OR

ps -aux | grep etcd
```

## Hands-on Lab: Manage etcd data securely using TLS Certificates (manually)

### Insert data into `etcd` securely

```
# Insert Data into etcd (manually)
sudo etcdctl --endpoints=https://10.0.0.10:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key put trainer "bindesh"
```

### Fetch data from `etcd` securely

```
# Get the key value for a Key (manually)

sudo etcdctl --endpoints=https://10.0.0.10:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key get trainer

```

etcdctl --endpoints=https://10.0.0.10:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key endpoint status --write-out=table

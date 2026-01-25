# kube-bench - Kubernetes CIS Benchmarking Tool

## 01. What is kube-bench?

- Kube-bench is a tool developed by _Aqua Security_, written in _GoLang_ that runs checks against the Center for Internet Security (CIS) Kubernetes Benchmark.
- The K8s CIS Benchmark is a set of security best practices for hardening Kubernetes clusters.

## 02. Reference

- GitHub Repository - https://github.com/aquasecurity/kube-bench
- CIS Benchmark for Kubernetes - https://www.cisecurity.org/benchmark/kubernetes
- kube-bench Releases - https://github.com/aquasecurity/kube-bench/releases

## 03. Lab: Run CIS Benchmark Assessment on Kubernetes cluster using kube-bench

### 3.1 Setup `kube-bench` as a CLI utility

#### Step-01: Connect to your control plane (master) node and create a new directory, say /opt/kube-bench

```
$ mkdir /opt/kube-bench
```

#### Step-02: Navigate to kube-bench releases page and download the latest linux binary file

```
$ curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.13.0/kube-bench_0.13.0_linux_amd64.tar.gz -o /opt/kube-bench/kube-bench.tar.gz

```

#### Step-03: Extract the downloaded kube-bench tarball to /opt/kube-bench directort

```
cd /opt/kube-bench
tar -xvf kube-bench.tar.gz

[Observe the extracted directed | It will have cfg directory with various benchmark and config files for k8s distros]

sudo mv /opt/kube-bench/kube-bench /usr/local/bin/
```

#### Step-04: Run the benchmark checks using kube-bench executable

```
kube-bench --config-dir cfg/ --config cfg/config.yaml
OR
kube-bench --config-dir cfg/ --config cfg/config.yaml > kube-bench.report
OR
kube-bench --benchmark cis-1.11 --config-dir cfg/
OR
kube-bench run master --benchmark cis-1.11 --config-dir cfg/


OR
# Run kube-bench in verbose mode
kube-bench run master -v 3 --benchmark cis-1.11 --config-dir cfg/

[The output will include detailed information about changes and potentially extensive debugging information along with the test results. The default output includes information logs, test results (PASS, FAIL, WARN, INFO), and remediations. ]
```

### 3.2 Running kube-bench on Kubernetes as a Pod (Job)

#### Step-01: Apply the kubernetes job for kube-bench

```
# Apply the k8s job manifest directly from the aquasecurity github repo
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

OR

# Download the manifest locally and then customize it (if required) >> then apply
curl https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml > job.yaml

kubectl apply -f job.yaml
```

#### Step-02: Get the kube-bench Job and Pod

```
# List all the Jobs | Observe the kube-bench Job
kubectl get job

# List all the Pods | Observe the kube-bench Pod
kubectl get pods

# Next, use the kube-bench pod name to get it's logs
kubectl logs <KUBE_BENCH_POD_NAME>

# To export the kube-bench pod logs to a file
kubectl logs <KUBE_BENCH_POD_NAME> > kube-bench.report
```

## 3.3 kube-bench - Common Errors

- **Error Message**

```
unable to determine benchmark version: config file is missing 'version_mapping' section
```

- **Description**: If you run the kube-bench command without providing the **--config-dir** and **--config** options, you will get the above error. So make sure you pass the values for both the parameters.

## 3.X Run the kube-bench with a particular cfg and k8s version

```
template:
    spec:
      hostPID: true
      containers:
      - name: kube-bench
        image: aquasec/kube-bench:latest
        command: ["kube-bench", "run", "--config-dir", "/etc/kube-bench/cfg", "--config", "/etc/kube-bench/cfg/config.yaml", "--version", "1.29"]
```

## 04. Lab: Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi)

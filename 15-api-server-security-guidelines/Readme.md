# kube-apiserver Security Guidelines

As per CIS Benchmarks for Kubernetes, following are the recommendations related to API Server:

- Ensure that the `--anonymous-auth` argument is set to false (Manual)
- Ensure that the `--token-auth-file` parameter is not set (Automated)
- Ensure that the DenyServiceExternalIPs is set (Manual)
- Ensure that the `--kubelet-client-certificate` and `--kubeletclient-key` arguments are set as appropriate (Automated)
- Ensure that the `--kubelet-certificate-authority` argument is set as appropriate (Automated)
- Ensure that the `--authorization-mode` argument is not set to AlwaysAllow (Automated)
- Ensure that the `--authorization-mode` argument includes Node (Automated)

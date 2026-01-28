# Securing the `Kubelet`

## Kubelet - Key Concepts

- kubelet by default operates at following ports:
  - **10250**
    - Serves API that allows full access
  - **10255**
    - Serves API that allows unauthenticated read-only acces

## Kubelet - Key configuration Files

- **/var/lib/kubelet/config.yaml**
  - The primary kubelet configuration file.
  - Alternate name /var/lib/kubelet/kubelet-config.yaml
- **/var/lib/kubelet/kubeadm-flags.env**
  - This environment file contains command-line flags (e.g., cgroup driver, CRI socket path) passed to the kubelet at startup.
- **/etc/kubernetes/kubelet.conf**
  - It contains the credentials (client certificates) the kubelet uses to communicate securely with the Kubernetes API server.
- **Service file (installed by package manager)**
  - **/usr/lib/systemd/system/kubelet.service**
  - It may vary slightly by OS, sometimes found in /lib/systemd/system/
- **kubeadm-specific systemd drop-in file**
  - /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf (for rpm-based systems like CentOS).
  - /etc/systemd/system/kubelet.service.d/10-kubeadm.conf (often used for Debian/Ubuntu based systems or administrator overrides).

### Viewing kubelet configuration

```
cat /var/lib/kubelet/config.yaml
```

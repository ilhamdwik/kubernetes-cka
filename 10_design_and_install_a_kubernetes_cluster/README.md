# Design and Install a Kubernetes Cluster

```
kube-controller-manager --leader-elect true 
                        --leader-elect-lease-duration 15s 
                        --leader-elect-renew-deadline 10s
                        --leader-elect-retry-period 2s
```

```
cat /etc/systemd/system/kube-apiserver.service
```

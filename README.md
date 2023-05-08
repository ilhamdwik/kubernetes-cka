# kubernetes-cka

### Cek Release
```
cat /etc/*release*
```


### Mengubah default namespace
kubectl config set-context $(kubectl config current-context) --namespace=(nama-namespace)


### Update Version Kubernetes

[Upgrade Version Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

```
kubectl drain (node-name)

sudo apt-get update

sudo apt-get install kubeadm=1.27.0

sudo apt-get install kubelet=1.27.0

kubeadm upgrade apply v1.27.0

sudo systemctl restart kubelet
```

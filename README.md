# kubernetes-cka


### if error The connection to the server 10.10.3.250:6443 was refused - did you specify the right host or port? on kubernetes

run script below
```
sudo systemctl restart kubelet

sudo systemctl restart containerd

sudo systemctl restart docker
```

```
cat .kube/config
```

if .kube/config null
```
mkdir -p $HOME/.kube

sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
crictl pods
```


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

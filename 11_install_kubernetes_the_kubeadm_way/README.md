# Install "Kubernetes the Kubeadm Way"

Resources
The vagrant file used in the next video is available here:

https://github.com/kodekloudhub/certified-kubernetes-administrator-course

Here's the link to the documentation:

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


### Demo deployment with kubeadm

kubernetes.io

##### Installing Kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

1. Container Runtimes
https://kubernetes.io/docs/setup/production-environment/container-runtimes/

####### install on master & worker node
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

```
lsmod | grep br_netfilter
lsmod | grep overlay
```

```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

install docker
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install containerd.io

systemctl status containerd
```

check cgroup drivers
```
ps -p 1
```

```
cd /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

sudo systemctl restart containerd

systemctl status containerd
```


2. Installing kubeadm, kubelet and kubectl

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


3. Creating a cluster with kubeadm

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

only on the master node
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=(ip-address-master-node)

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


4. Installing addons

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/

only on master node
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

and check pods
```
kubectl get pods -A
```

```
kubectl get daemonset -A
```

check cluster-cidr
```
kubectl cluster-info dump | grep -m 1 cluster-cidr
```

and edit into daemonset weave-net
```
kubectl edit daemonset weave-net -n kube-system

and edit

containers:
        - name: weave
          env:
            - name: IPALLOC_RANGE <-- add this line
              value: (cluster-cidr)  <-- add this line
```

check pod weave-net
```
kubectl get pods -A
```


5. Kubeadm Join - only on worker nodes

```
sudo kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

```
kubectl get pods

kubectl get nodes
```

# kubernetes-cka

### Exec Pod
```
kubectl exec -it (pod-name) -- /bin/bash
```

### Check Pod Network CIDR
```
kubectl cluster-info dump | grep -m 1 cluster-cidr
```

### Check Services IPs Range
```
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
```

### Check Pod Subnet & Service Subnet
```
kubectl describe cm kubeadm-config -n kube-system |grep Subnet
```

### Change default StorageClass
```
kubectl patch storageclass (storageclass-name) -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

kubectl patch storageclass (storageclass-name) -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Delete pod with status Evicted
```
kubectl get pod -n (namespace) | grep Evicted | awk '{print $1}' | xargs kubectl delete pod -n (namespace)
```

### Delete ReplicaSet with status 0
```
kubectl delete replicaset $(kubectl get replicaset -o jsonpath='{ .items[?(@.spec.replicas==0)].metadata.name }' -n (namespace)) -n (namespace)
```

### Delete namespace terminating
```
k get ns svc-prom -o json > tmp.json
vi tmp.json 
nano tmp.json

change "finalize=null"

k replace --raw "/api/v1/namespaces/svc-prom/finalize" -f tmp.json 
nano tmp.json 
k replace --raw "/api/v1/namespaces/svc-prom/finalize" -f tmp.json 
```

### cka study guide
[url : https://github.com/bmuschko/cka-study-guide](https://github.com/bmuschko/cka-study-guide)

### if error The connection to the server 10.10.3.250:6443 was refused - did you specify the right host or port? on kubernetes

##### run script below
```
sudo systemctl restart kubelet

sudo systemctl restart containerd

sudo systemctl restart docker
```

##### check config on kubernetes
```
cat .kube/config
```

##### if .kube/config null, run script bellow
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
```
kubectl config set-context $(kubectl config current-context) --namespace=(name-namespace)

OR

kubectl config set-context --current --namespace=(name-namespace)
```


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

### Count Words ClusterRoles
```
-w : count words
-c : count bytes
-m : count characters
-l : count newlines
-L : print longest line length

kubectl get clusterroles --no-headers | wc -w
```

### Get ipaddresspolls in metallb
```
kubectl get ipaddresspool.metallb.io -n metallb-system
```


### api-resources
```
kubectl api-resources
```


### if delete namespace and still terminating
```
kubectl get namespace <YOUR_NAMESPACE> -o json > <YOUR_NAMESPACE>.json

kubectl get namespace $NAMESPACE -o json   | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/"   | kubectl replace --raw /api/v1/namespaces/$NAMESPACE/finalize -f -

or

kubectl replace --raw "/api/v1/namespaces/<YOUR_NAMESPACE>/finalize" -f ./<YOUR_NAMESPACE>.json
```


### Port Kubernetes
![01](kubernetes_port.png)


--------------------------------------------------------------------------------------------------------


### Kubectx dan Kubens – Utilitas baris perintah
Sepanjang kursus, Anda harus mengerjakan beberapa ruang nama berbeda di lingkungan lab praktik. Di beberapa lab, Anda juga harus beralih di antara beberapa konteks.



Meskipun ini sangat bagus untuk praktik langsung, dalam kluster kubernetes “langsung” nyata yang diimplementasikan untuk produksi, mungkin ada kemungkinan untuk sering beralih antara sejumlah besar ruang nama dan kluster.



Ini dapat dengan cepat menjadi tugas yang membingungkan dan membebani jika Anda harus mengandalkan kubectl saja.



Di sinilah alat baris perintah seperti kubectx dan kubens masuk ke dalam gambar.



Referensi: https://github.com/ahmetb/kubectx



Kubectx:

Dengan alat ini, kamu tidak perlu menggunakan perintah "kubectl config" yang panjang untuk beralih antar konteks. Alat ini sangat berguna untuk mengalihkan konteks antar cluster dalam lingkungan multi-cluster.



Instalasi:
```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
```

Sintaksis:

Untuk membuat daftar semua konteks:
```
kubectx
```


Untuk beralih ke konteks baru:
```
kubectx <context_name>
```


Untuk beralih kembali ke konteks sebelumnya:
```
kubectx -
```


Untuk melihat konteks saat ini:
```
kubectx -c
```




Kubens:

Alat ini memungkinkan pengguna untuk beralih antar ruang nama dengan cepat menggunakan perintah sederhana.

Instalasi:
```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

Sintaksis:

Untuk beralih ke namespace baru:
```
kubens <new_namespace>
```


Untuk beralih kembali ke namespace sebelumnya:
```
kubens -
```

---------------------------------------------------------------------------------------------------------


         - RBAC (User can deploy etc)
         - Multi image/container on single pod
         - Upgrade kubernetes
         - PersistenVolume & PersistenVolumeClaim
         - statefulset
         - create svc 
         - create svc with response
         - busybox 
         - ingress
         - cordon & uncordon Node
         - node not running

---------------------------------------------------------------------------------------------------------


# Materi CKA

Domains & Competencies
Expand All

Storage 10%
```
Understand storage classes, persistent volumes
Understand volume mode, access modes and reclaim policies for volumes
Understand persistent volume claims primitive
Know how to configure applications with persistent storage
```

Troubleshooting 30%
```
Evaluate cluster and node logging
Understand how to monitor applications
Manage container stdout & stderr logs
Troubleshoot application failure
Troubleshoot cluster component failure
Troubleshoot networking
```

Workloads & Scheduling 15%
```
Understand deployments and how to perform rolling update and rollbacks
Use ConfigMaps and Secrets to configure applications
Know how to scale applications
Understand the primitives used to create robust, self-healing, application deployments
Understand how resource limits can affect Pod scheduling
Awareness of manifest management and common templating tools
```

Cluster Architecture, Installation & Configuration 25%
```
Manage role based access control (RBAC)
Use Kubeadm to install a basic cluster
Manage a highly-available Kubernetes cluster
Provision underlying infrastructure to deploy a Kubernetes cluster
Perform a version upgrade on a Kubernetes cluster using Kubeadm
Implement etcd backup and restore
```

Services & Networking 20%
```
Understand host networking configuration on the cluster nodes
Understand connectivity between Pods
Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
Know how to use Ingress controllers and Ingress resources
Know how to configure and use CoreDNS
Choose an appropriate container network interface plugin
```


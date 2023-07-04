# Upgrade Cluster Kubernetes

link : https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.27.x-00 && \
apt-mark hold kubeadm

kubeadm version

kubeadm upgrade plan
sudo kubeadm upgrade apply v1.27.x

kubectl drain <node-to-drain> --ignore-daemonsets


apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.27.x-00 kubectl=1.27.x-00 && \
apt-mark hold kubelet kubectl


sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon <node-to-uncordon>

kubectl get node -o wide



# Manage RBAC

link : https://kubernetes.io/docs/reference/access-authn-authz/rbac/

k create role -h 

kubectl create role foo --verb=get,list,watch --resource=pods

k create rolebinding -h

kubectl create rolebinding admin role=admin --user=user1

k create clusterrole -h

kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

k create clusterrolebinding -h

kubectl create clusterrolebinding cluster-admin --clusterrole=cluster-admin --user=user1

k auth can-i -h

k auth can-i get pod --as=user1

k auth can-i --list --as=system:serviceaccount:cka:kubernetes-dashboard-sa

kubectl auth can-i list pods --as=system:serviceaccount:dev:foo -n prod

kubectl auth can-i get pods --as=system:serviceaccount:cka:kubernetes-dashboard-sa



# cordon & uncordon node

k drain -h

kubectl drain <node>

k cordon -h

k cordon <node>

k uncordon -h

k uncordon <node>



# Multi Image / Multi Container

```
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:

  - name: alpha
    image: nginx
    env:
    - name: name
      value: "alpha"

  - name: beta
    image: busybox
    command: ["sleep", "4800"]
    env:
    - name: name
      value: "beta"
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-multi
  name: pod-multi
spec:
  containers:
  - image: nginx
    name: nginx
  - image: busybox
    name: busybox
status: {}
```



# Storage Classes & Persistent Volumes & Persistent Volume Claims

link : https://kubernetes.io/docs/concepts/storage/volumes/
link : https://kubernetes.io/docs/concepts/storage/persistent-volumes/
link : https://kubernetes.io/docs/concepts/storage/storage-classes/

### Storage Class
NFS
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-nfs
provisioner: external-nfs
parameters:
  #server: 10.10.3.249
  #path: /srv/nfs/mydata
  server: 10.10.3.250
  path: /data
  readOnly: "false"

Local
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer


### Persistent Volumes

 



# Ingress

link : 

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

initContainers
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container
  name: multi-container
spec:
  containers:
  - image: lisenet/httpd-pii-demo:0.2
    name: blue
    resources: {}
  - image: lisenet/httpd-healthcheck:1.0.0
    name: healthcheck
  initContainers:
  - name: busybox
    image: busybox:1.35.0
    command: ['sh', '-c', 'echo FIRST']
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

Hostpath
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: example-nfs
  hostPath:
    path: /data/ 
```


Local
link : https://kubernetes.io/docs/concepts/storage/volumes/#local

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```


### Persistent Volume Claims

link : https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumes-typed-hostpath

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
  storageClassName: example-nfs
```



# Ingress

link : https://kubernetes.io/docs/concepts/services-networking/ingress/

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-one  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-one
  template:
    metadata:
      labels:
        app: aks-helloworld-one
    spec:
      containers:
      - name: aks-helloworld-one
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"

---

apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-one  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-one
``` 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-two  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-two
  template:
    metadata:
      labels:
        app: aks-helloworld-two
    spec:
      containers:
      - name: aks-helloworld-two
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "AKS Ingress Demo"

---

apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-two  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-two
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - www.ingress-tls.com
    secretName: ingress-cert
  rules:
  - host: "www.ingress-tls.com"
    http:
      paths:
      - path: /hello-world-one(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-one
            port:
              number: 80
      - path: /hello-world-two(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-two
            port:
              number: 80
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-one
            port:
              number: 80
```



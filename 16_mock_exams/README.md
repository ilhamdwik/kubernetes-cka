# Mock Exams

Mock Exam - 1
Note: These tests are in beta/experimental phase as of now. Please report any issues/concerns through the slack channel or Q&A section.

These exams were built to give you a real exam like feel in terms of your ability to read and interpret a given question, validate your own work, manage time to complete given tasks within the given time, and see where you went wrong.

Having said that:

Please note that this exam is not a replica of the actual exam

Please note that the questions in these exams are not the same as in the actual exam

Please note that the interface is not the same as in the actual exam

Please note that the scoring system may not be the same as in the actual exam

Please note that the difficulty level may not be the same as in the actual exam



Mock Test Link - https://uklabs.kodekloud.com/topic/mock-exam-1-4/



This is the first of its kind. More on the way!

### Solution Mock Exam - 1

```
kubectl create ns exams

kubectl config set-context --current --namespace=exams


1.
kubectl run nginx-pod --image=nginx:alpine
kubectl get pod -w
kubectl get pod 
kubectl describe pod nginx-pod

2.
kubectl run pod -h
kubectl run pod labels -h
kubectl run messaging --image=redis:alpine --labels="tier=msg"
kubectl get pod -w
kubectl get pod

3.
kubectl create ns apx-x9984574
kubectl get ns

4.
kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json

5.
kubectl expose pod messaging --port=6379 --name=messaging-service
kubectl get pods -o wide
kubectl describe service messaging-service

6.
kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2

7.
kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml
cat static-busybox.yaml 
sudo mv static-busybox.yaml  /etc/kubernetes/manifests/static-busybox.yaml
kubectl get all -n default
kubectl describe pod/static-busybox-vmmaster -n default

8.
k create ns finance
k get ns
k run temp-bus --image=redis:alpine -n finance
k config set-context --current --namespace finance
k get all

9.
'error Init:CrashLoopBackOff'
k describe pod orange
k logs orange (nama-container)
k logs orange init-myservice
k edit pod orange
k replace --force -f (directory-file-yaml)

10.
k expose deploy hr-web-app --port=8080 --type=NodePort --name=hr-web-app-service 
k get all
k describe service/hr-web-app-service
k edit service/hr-web-app-service
edit "- nodePort: 30082"
k get all

11.
get name nodes & get osImage = kubectl get nodes -o=jsonpath="{.items[*]['metadata.name', 'status.nodeInfo.osImage']}"
get osImage                  = kubectl get nodes -o=jsonpath="{.items[*].status.nodeInfo.osImage}" 

12.
nano pv-analytics.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pv/data-analytics"


k create -f pv-analytics.yaml 
k get pv
k describe pv/pv-analytics
```


### Solution Mock Exam - 2
```
1. etcd backup / etcd snapshot
mkdir snapshot-etcd

ETCDCTL_API=3 etcdctl snapshot save -h

cat /etc/kubernetes/manifests/etcd.yaml | grep file

sudo ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   --cacert=/etc/kubernetes/pki/etcd/ca.crt   snapshot save snapshot-etcd/snapshot.db

cd snapshot-etcd/

cat snapshot.db

2. emptDir Volumes
nano pod-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-storage
spec:
  containers:
  - name: redis-storage
    image: redis:alpine
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}

k create -f pod-volume.yaml

k describe pod/redis-storage

3. set capabilities for a container

nano super-user-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: super-user-pod
spec:
  containers:
  - name: super-user-pod
    image: busybox:1.28
    command: ["sleep", "4800"]
    securityContext:
      capabilities:
        add: ["SYS_TIME"]

k create -f super-user-pod.yaml

4. use pv & pvc to pod
nano pv-1.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv-1

k create -f pv-1.yaml


nano pvc-1.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi

k create -f pvc-1.yaml

nano use-pv.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
      - mountPath: "/data"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc

k create -f use-pv.yaml

k get sc,pv,pvc

k get all

k describe pod use-pv

5. using RollingUpdate "update image nginx:1.16 to nginx:1.17"

k create deploy nginx-deploy --image=nginx:1.16 --replicas=1
k describe deploy nginx-deploy
k set image deploy nginx-deploy nginx=nginx:1.17
k describe deploy nginx-deploy

6.
openssl genrsa -out john.key 2048
openssl req -new -key john.key -subj "/CN=kube-developer" -out john.csr

cat john.csr | base64 | tr -d "\n" <-- Copy value to spec.request in john-scr.yaml

nano john-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1hqQ0NBVVlDQVFBd0dURVhNQlVHQTFVRUF3d09hM1ZpWlMxa1pYWmxiRzl3WlhJd2dnRWlNQTBHQ1>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

k create -f john-csr.yaml 

k get csr
check condition is pending 

k certificate approve john-developer

k get csr
check condition is approved

k create ns development

kubectl create role developer --verb=create,get,list,update,delete --resource=pods -n development

k config set-context --current --namespace=development

k get role

k describe role developer

k auth can-i -h

kubectl auth can-i create pods --as=john
no

k create rolebinding -h

kubectl create rolebinding john-developer --role=developer --user=john

k get role,rolebinding

k describe rolebinding.rbac.authorization.k8s.io/john-developer

kubectl auth can-i create pods --as=john
yes

kubectl auth can-i list pods --as=john
yes

kubectl auth can-i update pods --as=john
yes

kubectl auth can-i watch pods --as=john
no
```

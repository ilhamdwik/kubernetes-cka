# kubernetes


    | Kind          | Version      |
    |---------------|--------------|
    | POD           | v1           |
    | Service       | v1           |
    | ReplicaSet    | apps/v1      |
    | Deployment    | apps/v1      |




----------------------------------------------------------------------


# Imperative comment


# Create Pod and expose Service
kubectl run httpd --image=httpd:alpine --port=80 --expose=true


# Create Objects

kubectl run nginx --image=nginx

kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port 80 --target-port 80 --type NodePort

kubectl create -f nginx.yaml


# Update Objects

kubectl edit deployment nginx

kubectl scale deployment nginx replicas=5

kubectl set image deployment nginx nginx=nginx:1.18


# Replace (mengganti/memperbarui) Objects

kubectl replace -f nginx.yaml

kubectl replace --force -f nginx.yaml

 
# Delete Objects

kubectl delete -f nginx.yaml




# Declarative comment

kubectl apply -f nginx.yaml

kubectl apply -f /path/to/config-files


# See Manage Kubernetes Objects in kubernetes.io 
## link in below

[Manage Kubernetes Objects](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/)



----------------------------------------------------------------------
membuat pod redis dengan cara declarative

POD

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-containers
      image: nginx
    
----------------------------------------------------------------------
kubectl get pods
kubectl describe pod myapp-pod
kubectl apply -f pod.yaml

membuat pod redis dengan cara imperative
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
kubectl create -f redis.yaml

# 5

## Updates and Rollback

### Rollout Command
```
kubectl rollout status deployment/myapp-deployment
```

```
kubectl rollout history deployment/myapp-deployment
```

### kubectl apply
```
kubectl apply -f deployment-definition.yml
```

```
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1
```


---------------------------------------------------------------------------------------------------


## Commands & Arguments

#### pod-definition.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep"]
      args: ["1000"]
```

```
kubectl run webapp --image=nginx --command -- sleep 1000

kubectl run webapp --image=nginx --command -- python app2.py --color green

kubectl run webapp --image=nginx -- --color green
```

---------------------------------------------------------------------------------------------------


## ENV Variables in Kubernetes

pod-definition.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

### ENV Value Types

##### Plain Key Value
```
env:
  - name: APP_COLOR
    value:
```

##### ConfigMap
```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

##### Secrets
```
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```


---------------------------------------------------------------------------------------------------


## ConfigMap

#### Imperative

```
kubectl create configmap (configmap-name) --from-literal=(key)=(value)

kubectl create configmap app-config --from-literal=APP_COLOR=blue

kubectl create configmap app-config --from-literal=APP_MODE=prod
```

```
kubectl create configmap (configmap-name) --from-file=(path-to-file)

kubectl create configmap app-config --from-file=app_config.properties
``` 


#### Declarative

```
kubectl create -f configmap.yaml
```

configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```


#### ConfigMap in Pods

configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

pod-definition.yaml
```
apiVersion: vi
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

#### ENV
```
envFrom:
  - configMapRef:
      name: app-config
```

#### SINGLE ENV
```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

#### VOLUME
```
volumes:
  - name: app-config-volume
    configMap:
      name: app-color
```


---------------------------------------------------------------------------------------------------


### Create SECRET

Secret
DB_Host: mysql
DB_User: root
DB_Password: paswrd


#### Imperative
```
kubectl create secret generic (nama-secret) --from-literal=(key)=(value)

kubectl create secret generic app-secret --from-literal=DB_Host=mysql

kubectl create secret generic (nama-secret) --from-file=(path-to-file)

kubectl create secret generic (nama-secret) --from-file=app_secret.properties
```


#### Declarative

secret-data.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: paswrd
```

##### OR

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

##### untuk membuat data agar ter-encode dilinux jalankan command di bawah

```
echo -n 'mysql' | base64
echo -n 'root' | base64
echo -n 'paswrd' | base64
```

```
kubectl create -f secret-data.yaml
```


#### View Secret
```
kubectl get secret

kubectl describe secret

kubectl get secret app-secret -o yaml
```

##### untuk meng-encode data
```
echo -n 'bXlzcWw=' | base64 --decode
echo -n 'cm9vdA==' | base64 --decode
echo -n 'cGFzd3Jk' | base64 --decode
```


#### Secret in Pods

link : https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/

secret-data.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

pod-definition.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - image: simple-webapp-color
      name: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - secretRef:
            name: app-secret
```

##### ENV
```
envFrom:
  - secretRef:
      name: app-secret
```

##### SINGLE ENV

```
env:
  - name:DB_Password
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_Password
```


##### VOLUME
```
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```
```
ls /opt/app-secret-volumes

cat /opt/app-secret-volumes/DB_Password
```

```
etcdctl

sudo apt-get install -etcd-client
```

```
sudo ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/(nama-secret)
```

##### OR

```
sudo ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/(nama-secret) | hexdump -C
```

##### Configuration and determining whether encryption at rest is already enabled
```
ps -aux | grep kube-api | grep "encryption-provider-config"
```

##### Encrypt Your Data
enc.yaml
```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
      - configmaps
      - pandas.awesome.bears.example
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: aWxoYXNrYW0=
      - identity: {}
```


```
sudo mv enc.yaml /etc/kubernetes/enc/enc.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.10.30.4:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # <-- add this line
    volumeMounts:
    ...
    - name: enc                           # <-- add this line
      mountPath: /etc/kubernetes/enc      # <-- add this line
      readonly: true                      # <-- add this line
    ...
  volumes:
  ...
  - name: enc                             # <-- add this line
    hostPath:                             # <-- add this line
      path: /etc/kubernetes/enc           # <-- add this line
      type: DirectoryOrCreate             # <-- add this line
```

##### if connection error host port when kubectl get secret, run: 
```
crictl pods
```

```
kubectl create secret generic db-secret-2 --from-literal=key=ilhaskam --from-literal=key2=ilhaskam2
```

```
sudo ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/db-secret-2 | hexdump -C
```

##### Ensure all Secrets are encrypted
```
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

##### [See this link](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)


---------------------------------------------------------------------------------------------------


#### Multi Container Pods

yellow.yaml
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: yellow
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon
    command:
      - "sleep"
      - "1000"
  - image: redis
    name: gold
```


---------------------------------------------------------------------------------------------------


#### InitContainers

In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.



But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where initContainers comes in.



An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:


```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 

You can configure multiple such initContainers as well, like how we did for multi-containers pod. In that case each init container is run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```


Read more about initContainers here. And try out the upcoming practice test.

##### [See this link](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)


---------------------------------------------------------------------------------------------------


#### Self Healing Applications
Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.

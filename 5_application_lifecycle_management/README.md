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

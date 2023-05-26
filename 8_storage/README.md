# Storage

### PVC
Using PVCs in Pods
Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:


```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.


Reference URL: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes


##### Create StorageClass
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-nfs
provisioner: external-nfs
parameters:
  server: 10.10.3.249
  path: /srv/nfs/mydata
  readOnly: "false"
```

##### Create Persistent Volume

pv.yml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: example-nfs
  hostPath:
    path: /data
```

##### Create Persistent Volume Claims

pvc.yml
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

##### Create Test pvc for Pod

test-pvc-in-pod.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: pvc-for-pod
spec:
  volumes:
    - name: pvc-storage
      persistentVolumeClaim:
        claimName: local-pvc
  containers:
    - name: pod-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pvc-storage
```



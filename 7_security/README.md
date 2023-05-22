# Security 


### Accounts
```
kubectl create serviceaccount sa1

kubectl get serviceaccount
```


### TLS in Kubernetes

CERTIFICATE AUTHORITY (CA)
```
openssl genrsa -out ca.key 2048

openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

ADMIN USER
```
openssl genrsa -out admin.key 2048

openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr

openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

```
curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt
```

KUBE API SERVER
```
openssl genrsa -out apiserver.key 2048

openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr

openssl.cnf

openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt
```

/etc/kubernetes/pki/apiserver.crt
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```


Resource: Download Kubernetes Certificate Health Check Spreadsheet
I have uploaded the Kubernetes Certificate Health Check Spreadsheet here:

https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools

Feel free to send in a pull request if you improve it.




### Solution View Certification

certificate file kube-api server
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml

- --tls-cert-file=/etc/kubernetes/pki/apiserver.key
```

certificate file ETCD server
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml

- --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
```


### create certificate for user
```
kubectl get csr

kubectl certificate approve (certificate-name)

kubectl certificate deny (certificate-name)

kubectl delete csr (certificate-name)
```

### Solution Certificates API

create tls certificates
```
openssl genrsa -out ilham.key 2048

openssl req -new -key ilham.key -subj "/CN=ilham" -out ilham.csr
```

create base 64 on single line
```
cat ilham.csr | base64 -w 0
```

cat > ilham.yaml
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZUQ0NBVDBDQVFBd0VERU9NQXdHQTFVRUF3d0ZhV3hvWVcwd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQQpBNElCRHdBd2dnRUtBb0lCQVFEaTBJNDUyV2dVdkFSay81aTMxS3p5TWxCc2xZSkw3WVZZRGFZakdHWGpwa3ZGCjRlWlo0cklQRHcwV04rTlVnZldLRlA0bmtmWXg1bHhBaVJPZjJPdEVDVk81ZVNpUEJOUUNFb0x3SXhwMVU5OWUKeitycVVXcU8rMlU2aTZaVWhsczUwYVYzWUduMURLckVQUnk4aUpyZzB2TmNVelo0M3F2b25OZjZLK0tPYnJwZQpqRldORWppQzlkQXVMZkpIK1Mvbnhiem9HempGM2xVWTQ0RENsdU5uR3M0b3gvYUxyMUpJUjRPMmtQNkZySGN2Cm5ZTzRYNTVIbnc4TEcyMytCMDBxa1BkVSswSUw3UXh1Y0JUUU1CZ25JYTRzRXo3WmQwd3Zad2NFK0wxZGU1Y20KZ254cGJqWmJmMmlkeTBPZUN3b1czcWVISlZxUkJjbGpTQVdaczdFM0FnTUJBQUdnQURBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUFGNHpwWG4wV1kzVWU3RHo0dXRQZ1RiY080NERJZjdLRW9kcGVIb1ZPZHVUMHUzTXcvdis2CnViZUVaVTdKNExjakNNODVEUWlZb3pXZENSTngyL0NIWExNc2VXRk9ia0lkay9leldyOG03M0llNmNtT1VnbXEKZ2NsYW5RbG9vaEdXa242VnhTa1VpcGhxZk4rbFNZemU3OWtDVWpDTXdscmNnSHVERURZdENqQkdONm96eXRFcgpkV2dVRFRSNjdYamZjT0YzWGtlM0hQSnJQcDAxUnJ1RDhVUlg4cFdFc01pd1pDRkdDaFROUkFWbjVKTUkwOFdVCkFHcVVVRWJKbEFTaTREQnhqU2kyeU9lRXo3V1g4N3JzYjk5ZnNKMVExY3RsczFKa1lsOXR1MXExQWFIT3pMSXgKVEZYTk9kTGxEUk5qQ1N3RTh2dlVZd2VBMHNyN21WYkZQQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

```
kubectl create -f ilham.yaml

and 

kubectl get csr

and 

kubectl certificate approve ilham

and 

kubectl certificate deny ilham

and 

kubectl delete csr ilham
```


### KubeConfig
```
kubectl config -h

kubectl config view

kubectl config view --kubeconfig=my-custom-config
```


##### ~/my-kube-config
```
apiVersion: v1
kind: Config

clusters:
- name: kubernetes
  cluster:
    certificate-authority-data: 
    server: https://10.10.3.250:6443

- name: production
  cluster:
    certificate-authority-data: 
    server: https://10.10.3.250:6443

- name: development
  cluster:
    certificate-authority-data: 
    server: https://10.10.3.250:6443

contexts:
- name: kubernetes-admin@kubernetes
  context:
    cluster: kubernetes
    user: kubernetes-admin

- name: dev-user@development
  context:
    cluster: development
    user: dev-user

- name: prod-user@production
  context:
    cluster: production
    user: prod-user

users:
- name: kubernetes-admin
  user:
    client-certificate-data: 
    client-key-data: 

- name: dev-user
  user:
    client-certificate-data: 
    client-key-data: 

- name: prod-user
  user:
    client-certificate-data: 
    client-key-data: 

current-context: kubernetes-admin@kubernetes
preferences: {}

OR

apiVersion: v1
kind: Config

clusters:
- name: kubernetes
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.10.3.250:6443

- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.10.3.250:6443

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.10.3.250:6443

contexts:
- name: kubernetes-admin@kubernetes
  context:
    cluster: kubernetes
    user: kubernetes-admin

- name: dev-user@development
  context:
    cluster: development
    user: dev-user

- name: prod-user@production
  context:
    cluster: production
    user: prod-user

users:
- name: kubernetes-admin
  user:
    client-certificate: /etc/kubernetes/pki/ca.crt
    client-key: /etc/kubernetes/pki/ca.key

- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user.key

- name: prod-user
  user:
    client-certificate: /etc/kubernetes/pki/users/prod-user.crt
    client-key: /etc/kubernetes/pki/users/prod-user.key

current-context: kubernetes-admin@kubernetes
preferences: {}
```


```
kubectl config use-context developer

OR

kubectl config use-context research --kubeconfig ~/my/my-kube-config
```

Persistent Key/Value Store
We have already seen how to secure the ETCD key/value store using TLS certificates in the previous videos. Towards the end of this course, when we setup an actual cluster from scratch we will see this in action.



### Role Based Access Controls (RBAC)
```
kubectl get roles

kubectl get rolebindings

kubectl describe role (role-name)

kubectl describe rolebinding (name-rolebinding)
```


##### Check Access in kluster kubernetes
```
kubectl auth can-i create deployments

kubectl auth can-i delete nodes
```


##### Solutions RBAC
```
kubectl create role developer --verb=list,create,delete --resource=pods,deployments --namespace (name-namespace)

kubectl create rolebinding dev-user-binding --role=developer --user=dev-user --namespace (name-namespace)

kubectl --as dev-user get pod (pod-name) -n (name-namespace)

kubectl --as dev-user create deployment nginx --image=nginx --namespace (name-namespace) 
```


### ClusterRoles & Role Bindings

melihat daftar lengkap sumber daya namespace dan non-namespace
```
kubectl api-resources --namespace=true

kubectl api-resources --namespace=false
```


##### create clusterroles & clusterrolebindings
```
kubectl create clusterrole michelle-role --verb=get,list,watch --resource=nodes

kubectl create clusterrolebinding michelle-role-binding --clusterrole=michelle-role --user=michelle

kubectl delete clusterrole michelle-role

kubectl delete clusterrolebinding michelle-role-binding
```

```
kubectl create clusterrole storage-admin --verb=get,list,watch,create,delete --resource=persistentvolumes,storageclasses

kubectl create clusterrolebinding michelle-storage-admin --user=michelle --clusterrole=storage-admin
```


### Service Account 
```
kubectl create serviceaccount (nama-serviceaccount)

kubectl create serviceaccount dashboard-sa

kubectl describe serviceaccount dashboard-sa

kubectl describe secret dashboard-sa-token-...

kubectl exec -it dashboard-sa --ls /var/run/secrets/kubernetes.io/serviceaccount

kubectl exec -it dashboard-sa ls /var/run/secrets/kubernetes.io/serviceaccount

kubectl exec -it dashboard-sa cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

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

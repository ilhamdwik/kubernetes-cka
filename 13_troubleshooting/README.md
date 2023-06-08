# Troubleshooting

### Application Failure

Check Accessibility

```
curl http://web-service-ip:node-port
```

Check Service Status
```
kubectl describe service web-service
```

Check Pod
```
kubectl get pod

kubectl describe pod web

kubectl logs web

kubectl logs web -f --previous
```

Check Dependent Service
DB Service

Check Dependent Applications
https://kubernetes.io/docs/tasks/debug/debug-application/


### Solution Application Failure
```
cek nama db_host, db_user, and db_password on deployment web and mysql. 
cek environment on pod
cek port and endpoints antara deployment dan service
cek label dan selector antara deployment dan service
```


### Control Plane Failure

https://kubernetes.io/docs/tasks/debug/debug-cluster/

Check Control Plane Pods
```
kubectl get nodes

kubectl get pods

kubectl get pods -n kube-system

service kube-apiserver status

service kube-controller-manager status

service kube-scheduler status

service kubelet status

service kube-proxy status
```

Check Service Logs
```
kubectl logs kube-apiserver-master -n kube-system

sudo kournalctl -u kube-apiserver
```


### Solution Control Plane Failure

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

kubectl autocomplete
```
source <(kubectl completion bash)

echo "source <(kubectl completion bash)" >> ~/.bashrc

alias k=kubectl

complete -o default -F __start_kubectl k
```

replicaset
```
/etc/kubernetes/controller-manager.conf
```

```
vi /etc/kubernete/manifests/kube-controller-manager.yaml

/etc/kubernetes/pki
```


### Worker Node Failure

Check Node Status

```
kubectl get nodes

kubectl describe node worker-1
```

Check Node
```
top

df -h
```

Check Kubelet Status
```
systemctl status kubelet

sudo journalctl -u kubelet
```

Check Certificates
```
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
```


### Solution Worker Node Failure

```
/var/lib/kubelet

/var/lib/kubelet/config.yaml

/etc/kubernetes/kubelet.conf
```

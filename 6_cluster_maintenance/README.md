# 6


## OS Upgrades
```
kubectl drain node-1

kubectl cordon node-1

kubectl uncordon node-1
```

OR

```
kubectl drain node-1

apt-get upgrade -y kubeadm=1.27.0-00

apt-get upgrade -y kubelet=1.27.0-00

kubeadm upgrade node config --kubelet-version v1.27.0

systemctl restart kubelet

kubectl cordon node-1

kubectl uncordon node-1

sudo kubeadm upgrade plan

sudo kubeadm apply v1.27.1
```


## References

https://kubernetes.io/docs/concepts/overview/kubernetes-api/

Here is a link to kubernetes documentation if you want to learn more about this topic (You don't need it for the exam though):

https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md

https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md



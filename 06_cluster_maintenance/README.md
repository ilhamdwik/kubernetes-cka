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


## Backup - Resource Configs
```
kubectl get all --all-namespaces -o yaml > all-deploy.yaml
```


## Working with ETCDCTL


etcdctl is a command line client for etcd.


In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.



You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:

export ETCDCTL_API=3

On the Master Node:
```
export ETCDCTL_API=3

etcdctl version
```


To see all the options for a specific sub-command, make use of the -h or --help flag.



For example, if you want to take a snapshot of etcd, use:

etcdctl snapshot save -h and keep a note of the mandatory global options.



Since our ETCD database is TLS-Enabled, the following options are mandatory:

--cacert : verify certificates of TLS-enabled secure servers using this CA bundle

--cert : identify secure client using this TLS certificate file

--endpoints=[127.0.0.1:2379] : This is the default as ETCD is running on master node and exposed on localhost 2379.

--key : identify secure client using this TLS key file



Similarly use the help option for snapshot restore to see all available options for restoring the backup.

etcdctl snapshot restore -h

For a detailed explanation on how to make use of the etcdctl command line tool and work with the -h flags, check out the solution video for the Backup and Restore Lab.


## Backup & Restore
version etcd
```
k describe pod (etcd-name) -n kube-system

version etcd is Image
```

```
address can you reach the ETCD cluster from the controlplane node
--listen-client-urls=https://127.0.0.1:2379
```

```
where is the ETCD server certificate file located?
--cert-file=/etc/kubernetes/pki/etcd/server.crt
```

```
where is the ETCD CA Certificate file located?
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

```
ETCDCTL_API=3 etcdctl snapshot

export ETCDCTL_API=3

etcdctl snapshot
```

### Backup ETCD to /opt/snapshot-pre-boot.db
```
etcdctl snapshot save --endpoints=127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
/opt/snapshot-pre-boot.db

OR

ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot-1.db
```

```
etcdctl snapshot restore --data-dir (hostpath) (path-file-backup)

etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot-1.db

ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot-1.db --data-dir /var/lib/etcd-from-backup
```

how many cluster?
```
kubectl config view
```

switch cluster
```
kubectl config use-context (cluster-name)
```

```
ps -ef | grep -i etcd
```

etcdctl member list
```
sudo ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key member list
```

```
vi /etc/systemd/system/etcd.service

sudo systemctl daemon-reload

sudo systemctl restart etcd

systemctl status etcd
```


### Certification Exam Tip!
Here's a quick tip. In the exam, you won't know if what you did is correct or not as in the practice tests in this course. You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the the kubectl describe pod command to verify the pod is created with the correct name and correct image.


### References
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

https://github.com/etcd-io/website/blob/main/content/en/docs/v3.5/op-guide/recovery.md

https://www.youtube.com/watch?v=qRPNuT080Hk

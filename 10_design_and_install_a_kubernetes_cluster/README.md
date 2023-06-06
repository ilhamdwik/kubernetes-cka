# Design and Install a Kubernetes Cluster

```
kube-controller-manager --leader-elect true 
                        --leader-elect-lease-duration 15s 
                        --leader-elect-renew-deadline 10s
                        --leader-elect-retry-period 2s
```

```
cat /etc/systemd/system/kube-apiserver.service
```


Important Update: Kubernetes the Hard Way
Installing Kubernetes the hard way can help you gain a better understanding of putting together the different components manually.

An optional series on this is available at our youtube channel here:



https://www.youtube.com/watch?v=uUupRagM7m0&list=PL2We04F3Y_41jYdadX55fdJplDvgNGENo



The GIT Repo for this tutorial can be found here: https://github.com/mmumshad/kubernetes-the-hard-way

# Networking

```
cat /proc/sys/net/ipv4/ip_forward

value 0 or 1
```

```
nano /etc/sysctl.conf

net.ipv4.ip_forward = 1
```

### catatan
```
ip link = membuat daftar & memodifikasi antarmuka pada host

ip addr = melihat alamat ip yang ditetapkan antarmuka

ip addr add 192.168.1.10/24 dev eth0 = mengatur alamat ip pada antarmuka

ip route & route = untuk melihat tabel routing dan rute ip

ip route = menambahkan entri ke dalam tabel routing

nb : perubahan ini hanya berlaku sampai virtual machine di restart
```


## DNS
Setting local DNS in system
```
nano /etc/hosts
```

Setting DNS Server
```
nano /etc/resolv.conf
```

check urutan membaca DNS
jika hosts: files dns
maka yang pertama kali dibaca adalah file /etc/hosts dilocal,jika tidak menemukan nama dns yang benar, host selanjutnya akan membaca file dns di server. 
```
cat /etc/nsswitch.conf
```

```
nslookup www.google.com

dig www.google.com
```


### CoreDNS
Prerequisite - CoreDNS
In the previous lecture we saw why you need a DNS server and how it can help manage name resolution in large environments with many hostnames and Ips and how you can configure your hosts to point to a DNS server. In this article we will see how to configure a host as a DNS server.



We are given a server dedicated as the DNS server, and a set of Ips to configure as entries in the server. There are many DNS server solutions out there, in this lecture we will focus on a particular one – CoreDNS.



So how do you get core dns? CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.






Run the executable to start a DNS server. It by default listens on port 53, which is the default port for a DNS server.



Now we haven’t specified the IP to hostname mappings. For that you need to provide some configurations. There are multiple ways to do that. We will look at one. First we put all of the entries into the DNS servers /etc/hosts file.



And then we configure CoreDNS to use that file. CoreDNS loads it’s configuration from a file named Corefile. Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the Ips and names from the /etc/hosts file on the server.




CoreDNS also supports other ways of configuring DNS entries through plugins. We will look at the plugin that it uses for Kubernetes in a later section.

Read more about CoreDNS here:

https://github.com/kubernetes/dns/blob/master/docs/specification.md

https://coredns.io/plugins/kubernetes/


### Create Network NameSpace

##### Exec in Network NS
```
ip netns add red

ip netns add blue

ip netns

ip netns exec red ip link = ip -n red link

arp

ip netns exec red arp

route

ip netns exec red route

ip link add veth-red type veth peer name veth-blue

ip link set veth-red netns red

ip link set veth-blue netns blue 

ip -n red addr add 192.168.15.1 dev veth-red

ip -n blue addr add 192.168.15.2 dev veth-blue

ip -n red link set veth-red up

ip -n blue link set veth-blue up 

ip netns exec red ping 192.168.15.2

ip netns exec red arp

ip netns exec blue arp

arp
```

##### Linux Bridge & Open vSwitch
```
ip link add v-net-0 type bridge

ip link

ip link set dev v-net-0 up

ip -n red link del veth-red

ip link add veth-red type veth peer name veth-red-br

ip link add veth-blue type veth peer name veth-blue-br 
```


```
ip link set veth-red netns red

ip link set veth-red-br master v-net-0

ip link set veth-blue netns blue

ip link set veth-blue-br master v-net-0

ip -n red addr add 192.168.15.1 dev veth-red

ip -n blue addr add 192.168.15.2 dev veth-blue

ip -n red link set veth-red up

ip -n blue link set veth-blue up

ping 192.168.15.1
Not Reachable!

ip addr add 192.168.15.5/24 dev v-net-0

ping 192.168.15.1
```

```
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE

iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
```


### FAQ
While testing the Network Namespaces, if you come across issues where you can't ping one namespace from the other, make sure you set the NETMASK while setting IP Address. ie: 192.168.1.10/24



ip -n red addr add 192.168.1.10/24 dev veth-red



Another thing to check is FirewallD/IP Table rules. Either add rules to IP Tables to allow traffic from one namespace to another. Or disable IP Tables all together (Only in a learning environment).



### Important Note about CNI and CKA Exam
An important tip about deploying Network Addons in a Kubernetes cluster.



In the upcoming labs, we will work with Network Addons. This includes installing a network plugin in the cluster. While we have used weave-net as an example, please bear in mind that you can use any of the plugins which are described here:

https://kubernetes.io/docs/concepts/cluster-administration/addons/

https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model



In the CKA exam, for a question that requires you to deploy a network addon, unless specifically directed, you may use any of the solutions described in the link above.



However, the documentation currently does not contain a direct reference to the exact command to be used to deploy a third party network addon.

The links above redirect to third party/ vendor sites or GitHub repositories which cannot be used in the exam. This has been intentionally done to keep the content in the Kubernetes documentation vendor-neutral.

At this moment in time, there is still one place within the documentation where you can find the exact command to deploy weave network addon:



https://v1-22.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node (step 2)



### Solution Explore Environment
```
ip address

ip address show type bridge

ip route
```


### CNI in Kubernetes

##### View kubelet options
```
ps -aux | grep kubelet

ls /opt/cni/bin

ls /etc/cni/net.d

sudo su & cat /etc/cni/net.d/10-calico
```


### Note CNI Weave
Important Update: -

Before going to the CNI weave lecture, we have an update for the Weave Net installation link. They have announced the end of service for Weave Cloud.

To know more about this, read the blog from the link below: -

https://www.weave.works/blog/weave-cloud-end-of-service

As an impact, the old weave net installation link won’t work anymore: -

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

Instead of that, use the below latest link to install the weave net: -

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Reference links: -

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation

https://github.com/weaveworks/weave/releases


### Solution Explore CNI

Check container runtime in kubelet
```
ps -aux | grep -i kubelet | grep -i container-runtime
```

Path configured with all binaries of CNI supported plugins
```
/opt/cni/bin
```

CNI plugin configured on kubernetes-cluster
```
/etc/cni/net.d
```


### Solution Deploy Network
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

OR

wget https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

edit weave-daemonset-k8s.yaml file and change value to cluster-cidr
```
containers:
  - name: weave
    env:
      - name: IPALLOC_RANGE
        value: 10.0.0.0/16 <--- change value in here with number cluster-cidr
```

to get cluster-cidr
```
k describe cm kube-proxy -n kube-system
```
and search clusterCIDR



### Solution Networking Weave

default gateway configured on the pods scheduled on vmworker1
```
kubectl run busybox --image=busybox --dry-run=client -o yaml -- sleep 1000 > busybox.yaml
```

edit nodeName:vmworker1
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  nodeName: vmworker1
  containers:
  - args:
    - sleep
    - "1000"
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```
kubectl create -f busybox.yaml
```

and check default gateway on pods busybox
```
kubectl exec busybox -- route -n

OR

kubectl exec busybox -- ip route
```


### Solution Service Networking

what network range are the nodes in the cluster part of?
is: 10.10.3.0/24

```
kubectl get nodes -o wide
10.10.3.250

ip address
10.10.3.250/24
```


what is the range of IP addresses configured for PODs on this cluster?
is: ipalloc-range:.....

```
kubectl logs (name-pod-weave) -n kube-system

and search "ipalloc-range:....."
```


what is the IP range configured for the services within the cluster?
is: - --service-cluster-ip-range=10.96.0.0/12

```
cd /etc/kubernetes/manifest/

cat kube-apiserver.yaml

and search "- --service-cluster-ip-range=10.96.0.0/12"
```


### Explore DNS
Identify DNS solution in this cluster
```
kubectl get pods -n kube-system

kubectl get pods -n kube-system

/etc/coredns/Corefile

kubectl exec (pod-name) -- nslookup (pod-name.name-namespace)

kubectl exec hr -- nslookup mysql.payroll
```


### Ingress
Article: Ingress
As we already discussed Ingress in our previous lecture. Here is an update.

In this article, we will see what changes have been made in previous and current versions in Ingress.

Like in apiVersion, serviceName and servicePort etc.

![01](ingress.png)

Now, in k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-

Format - kubectl create ingress <ingress-name> --rule="host/path=service:port"

Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

Find more information and examples in the below reference link:-

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-

References:-

https://kubernetes.io/docs/concepts/services-networking/ingress

https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types

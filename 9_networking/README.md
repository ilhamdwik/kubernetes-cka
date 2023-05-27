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

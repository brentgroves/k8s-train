https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
https://www.magalix.com/blog/kubernetes-cluster-networking-101


Introduction
Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures the kubelets to tell individual containers to use the DNS Service's IP to resolve DNS names.

Every Service defined in the cluster (including the DNS server itself) is assigned a DNS name. By default, a client Pod's DNS search list includes the Pod's own namespace and the cluster's default domain.

Namespaces of Services
A DNS query may return different results based on the namespace of the pod making it. DNS queries that don't specify a namespace are limited to the pod's namespace. Access services in other namespaces by specifying it in the DNS query.

For example, consider a pod in a test namespace. A data service is in the prod namespace.

A query for data returns no results, because it uses the pod's test namespace.

A query for data.prod returns the intended result, because it specifies the namespace.

DNS queries may be expanded using the pod's /etc/resolv.conf. Kubelet sets this file for each pod. For example, a query for just data may be expanded to data.test.svc.cluster.local. The values of the search option are used to expand queries. To learn more about DNS queries, see the resolv.conf manual page.

nameserver 10.32.0.10
search <namespace>.svc.cluster.local svc.cluster.local cluster.local
options ndots:5

In summary, a pod in the test namespace can successfully resolve either data.prod or data.prod.svc.cluster.local.

DNS Records
What objects get DNS records?

Services
Pods
The following sections detail the supported DNS record types and layout that is supported. Any other layout or names or queries that happen to work are considered implementation details and are subject to change without warning. For more up-to-date specification, see Kubernetes DNS-Based Service Discovery.

Services
A/AAAA records
"Normal" (not headless) Services are assigned a DNS A or AAAA record, depending on the IP family of the service, for a name of the form my-svc.my-namespace.svc.cluster-domain.example. This resolves to the cluster IP of the Service.

"Headless" (without a cluster IP) Services are also assigned a DNS A or AAAA record, depending on the IP family of the service, for a name of the form my-svc.my-namespace.svc.cluster-domain.example. Unlike normal Services, this resolves to the set of IPs of the pods selected by the Service. Clients are expected to consume the set or else use standard round-robin selection from the set.

SRV records
SRV Records are created for named ports that are part of normal or Headless Services. For each named port, the SRV record would have the form _my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster-domain.example. For a regular service, this resolves to the port number and the domain name: my-svc.my-namespace.svc.cluster-domain.example. For a headless service, this resolves to multiple answers, one for each pod that is backing the service, and contains the port number and the domain name of the pod of the form auto-generated-name.my-svc.my-namespace.svc.cluster-domain.example.

Pods
A/AAAA records
In general a pod has the following DNS resolution:

pod-ip-address.my-namespace.pod.cluster-domain.example.

For example, if a pod in the default namespace has the IP address 172.17.0.3, and the domain name for your cluster is cluster.local, then the Pod has a DNS name:

172-17-0-3.default.pod.cluster.local.

Any pods exposed by a Service have the following DNS resolution available:

pod-ip-address.service-name.my-namespace.svc.cluster-domain.example.

Pod's hostname and subdomain fields
Currently when a pod is created, its hostname is the Pod's metadata.name value.

The Pod spec has an optional hostname field, which can be used to specify the Pod's hostname. When specified, it takes precedence over the Pod's name to be the hostname of the pod. For example, given a Pod with hostname set to "my-host", the Pod will have its hostname set to "my-host".

The Pod spec also has an optional subdomain field which can be used to specify its subdomain. For example, a Pod with hostname set to "foo", and subdomain set to "bar", in namespace "my-namespace", will have the fully qualified domain name (FQDN) "foo.bar.my-namespace.svc.cluster-domain.example".



Link 2 (enp0s31f6)
      Current Scopes: DNS           
DefaultRoute setting: yes           
       LLMNR setting: yes           
MulticastDNS setting: no            
  DNSOverTLS setting: no            
      DNSSEC setting: no            
    DNSSEC supported: no            
  Current DNS Server: 10.1.2.69     
         DNS Servers: 10.1.2.69     
                      10.1.2.70     
                      172.20.0.39   
          DNS Domain: ~.            
                      BUSCHE-CNC.COM


https://microk8s.io/docs/addon-dns  
microk8s kubectl -n kube-system edit configmap/coredns 

microk8s kubectl -n kube-system describe configmap/coredns 
kubectl describe configmaps coredns -n kube-system
kubectl describe configmaps coredns -n kube-system -o yaml

After some quick debugging with kubectl exec -it, I discovered that the Pod/container did actually have access to the internet, because I could ping public IP addresses. (Thank you CAP_NET_RAW.)

If you do run systemd-resolve --status, it lists a whole bunch of stuff you probably don't care about, including what your real upstream DNS servers are.


Link 2 (enp0s31f6)
      Current Scopes: DNS           
DefaultRoute setting: yes           
       LLMNR setting: yes           
MulticastDNS setting: no            
  DNSOverTLS setting: no            
      DNSSEC setting: no            
    DNSSEC supported: no            
  Current DNS Server: 10.1.2.69     
         DNS Servers: 10.1.2.69     
                      10.1.2.70     
                      172.20.0.39   
          DNS Domain: ~.            
                      BUSCHE-CNC.COM


https://microk8s.io/docs/addon-dns                      

An endpoint is an resource that gets IP addresses of one or more pods dynamically assigned to it, along with a port. An endpoint can be viewed using kubectl get endpoints .Oct 17, 2018
he /etc/resolv.conf in my pods looks like this:

dig www.google.com

search default.svc.cluster.local svc.cluster.local cluster.local xxx.xxxxx
nameserver 10.152.183.10
options ndots:5
when I look at the logs of my kube-dns with $ microk8s kubectl logs --namespace=kube-system -l k8s-app=kube-dns I get the following response:

[INFO] 10.1.107.105:47549 - 5288 "AAAA IN www.google.com. udp 36 false 512" NOERROR - 0 0.000256103s
[ERROR] plugin/errors: 2 www.google.com. AAAA: read udp 10.1.107.127:51486->x.x.x.101:53: read: no route to host
DNS service is up:

$ microk8s kubectl get svc --namespace=kube-system
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
[...]
kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   21d


https://serverfault.com/questions/1078271/trouble-with-dns-resolution-on-and-microk8s-cluster

This deploys CoreDNS to supply address resolution services to
Kubernetes.

This service is commonly required by other addons, so it is
recommended that you enable it.

microk8s enable dns
By default it points to Google’s 8.8.8.8 and 8.8.4.4 servers for resolving
addresses. This can be changed when you enable the addon, for example:

microk8s enable dns:1.1.1.1
(for multiple DNS addresses, a comma-separated list should be used)

The forward dns servers can also be altered after enabling the addon
by running the command:

microk8s kubectl -n kube-system edit configmap/coredns
This will invoke the vim editor so that you can alter the configuration.

The addon can be disabled at any time:

microk8s disable dns
…but bear in mind this could have implications for services and pods which
may be relying on it.



https://blog.yaakov.online/kubernetes-getting-pods-to-talk-to-the-internet/#:~:text=If%20you%20install%20this%20with,make%20connections%20to%20the%20Internet.

After some quick debugging with kubectl exec -it, I discovered that the Pod/container did actually have access to the internet, because I could ping public IP addresses. (Thank you CAP_NET_RAW.)

The problem was in DNS.

After a couple hours of further debugging, I discovered the following:

All of my Kubernetes servers/VMs are running Ubuntu Server 17.10. Ubuntu uses systemd, and whilst I don't know much about systemd, I do know that it is heavily derided for being the exact opposite of the do-one-thing-and-do-it-well UNIX tooling philosphy.

In this case, systemd has it's own DNS resolver called systemd-resolved. This replaces the traditional resolver, and /etc/resolv.conf redirects applications to use this instead. resolv.conf looks like this (below).

If you do run systemd-resolve --status, it lists a whole bunch of stuff you probably don't care about, including what your real upstream DNS servers are.


Link 2 (enp0s31f6)
      Current Scopes: DNS           
DefaultRoute setting: yes           
       LLMNR setting: yes           
MulticastDNS setting: no            
  DNSOverTLS setting: no            
      DNSSEC setting: no            
    DNSSEC supported: no            
  Current DNS Server: 10.1.2.69     
         DNS Servers: 10.1.2.69     
                      10.1.2.70     
                      172.20.0.39   
          DNS Domain: ~.            
                      BUSCHE-CNC.COM


https://microk8s.io/docs/addon-dns                      

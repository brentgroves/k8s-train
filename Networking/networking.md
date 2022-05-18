https://www.magalix.com/blog/kubernetes-cluster-networking-101
What Is Cluster Networking In Kubernetes Sense?
Kubernetes is a technology that helps you get the most out of your hardware. Containers are deployed on several nodes, making sure that every CPU cycle, every byte of memory, and every block of storage is not wasted. However, this is no easy task. Several challenges must be addressed when designing how the cluster will handle networking among containers:

How the same pod containers can communicate with each other. This is handled through the loopback interface. For more information, please refer to our earlier article Kubernetes pods 101.
How a container can contact a service. This is /managed through Kubernetes Services.
How the cluster can receive external traffic. This topic is also covered by Kubernetes Services.
How a pod can contact another pod on the same node or on a different one, which is the main focus of this article.

Kubernetes Networking Rules
Kubernetes is a highly modular, open-source project. Several components were left to the community to develop. In particular, implementing a cluster-networking solution must conform to a set of high-level rules. They can be summarized as follows:

Pods scheduled on the same node must be able to communicate with other pods without using NAT (Network Address Translation).
All system daemons (background processes, for example, kubelet) running on a particular node can communicate with the pods running on the same node.
Pods that use the host network must be able to contact all other pods on all other nodes without using NAT. Notice that the host network is only supported on Linux hosts.

Network address translation is a method of mapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device.
Use host networking
If you use the host network mode for a container, that container’s network stack is not isolated from the Docker host (the container shares the host’s networking namespace), and the container does not get its own IP-address allocated. For instance, if you run a container which binds to port 80 and you use host networking, the container’s application is available on port 80 on the host’s IP address.

To appreciate the simplicity of this design, let’s see how we can manually create a number of containers (using Docker, for example) that will be distributed on a number of physical hosts, and how they can communicate without the Kubernetes design. First, you’ll need to use NAT to ensure that no port collision happens when more than one container tries to use the same port. Let’s say two Apache containers, both are running on port 80. None of those containers can receive traffic by exposing port 80 on the host; as a port collision will occur. This is only possible through NAT. Using NAT means that the container will not communicate through its own IP address or port. Rather, its IP will be hidden behind the NAT IP, and a unique port on the NAT interface (for example, 8080) will forward traffic to port 80 on the container. The second container will use the same NAT IP but with a different port and so on. The following graph depicts how Kubernetes implements its networking model versus the traditional way.

So, as you can see Kubernetes eliminates the need for NAT or link containers.

There are a number of networking models that adhere to the above rules. In this article, we’ll select some of them for discussion. But, before listing the different network plugin examples, let’s have a quick overview of some important Kubernetes networking terms.

What is an Overlay Network?
In general, we can define networks as underlay and overlay types:

Underlay network
The Underlay network is closer to the physical layer. It includes switches, routers, VLANs, and so on. It is the basis on which overlay networks are built. It tends to be less scalable due to technical limitations. However, since it’s closer to the actual hardware, it is slightly faster than an overlay.

A virtual local area network is any broadcast domain that is partitioned and isolated in a computer network at the data link layer. LAN is the abbreviation for local area network and in this context virtual refers to a physical object recreated and altered by additional logic
The data link layer, or layer 2, is the second layer of the seven-layer OSI model of computer networking. This layer is the protocol layer that transfers data between nodes on a network segment across the physical layer.

Overlay network
Overlay network refers to the virtual network layer. In this type, you’ll hear terms like veth (virtual eth or virtual network interface), and VxLAN. It is designed to be highly scalable than the underlying network. For example, while VLANs in the underlying network support only 4096 identifiers, VxLAN can reach up to 16 million ones.

Kubernetes supports both networking models, so you can base your model of choice on other factors than whether or not the cluster can handle it.

What is a Container Network Interface (CNI)?
A CNI is simply a link between the container runtime (like Docker or rkt) and the network plugin. The network plugin is nothing but the executable that handles the actual connection of the container to or from the network, according to a set of rules defined by the CNI. So, to put it simply, a CNI is a set of rules and Go libraries that aid in container/network-plugin integration.

All of the CNIs can be deployed by simply running a pod or a daemonset that launches and manages their daemons. Let’s have a look now at the most well-known Kubernetes networking solutions.

Calico
Calico is a scalable and secure networking plugin. It can be used to manage and secure network policies not only for Kubernetes, but also for containers, virtual machines, and even bare metal servers. Calico works on Layer 3 of the network stack. It works by implementing a vRouter (as opposed to a vSwitch) on each node. Since it is working on L3, it can easily use the Linux kernel’s native forwarding functionality. The Felix agent is responsible for programming L3 Forwarding Information base with the IP addresses of the pods scheduled on the node where it is running.

Calico uses vRouters to allow pods to connect to each other across different nodes using the physical network (underlay). It does not use overlay, tunneling or VRF tables. Also, it does not require NAT since each pod can be assigned a public IP address that is accessible from anywhere as long as the security policy permits it.

Deployment differs based on the type of environment or the cloud provider where you’ll be hosting your cluster. This document contains all the supported Calico deployment methods.


Cilium
Cilium uses layers 3, 4 (network), and layer 7 (application) to function. It brings a solution that is not only aware of the packets that pass through, but also the application and protocol (for example, HTTP) that those packets are using. Having such a level of inspection allows Cilium to control and enforce network and application security policies. Be aware, though that for this plugin to work, you must be using a Linux kernel that is equal to or higher than 4.8. That’s because Cilium uses a new kernel feature Berkeley Packet Filter (BPF), which can replace iptables.

Cilium runs a daemon called cilium-agent on each node. It compiles the BPF filters and transfers them to the kernel for further processing.

For different ways to install Cilium, which includes using your own machine (through minikube or microk8s), please follow this guide.

Weave Net from WeaveWorks
Weave Net is an easy-to-use, resilient, and fast-growing network plugin that can be used for more than just container networking. When installed, Weave Net creates a virtual router on each host (called peer). Those routers start communicating with each other to establish a protocol handshake and, later, learn the network topology. The plugin also creates a bridge interface on each host. All pods get attached to this interface, and they are assigned IP addresses and netmasks. Within the same node, Weave Net uses the kernel to move packets from one pod to another. This protocol is called the fast data path. When the packet is destined to a pod on another host, the plugin uses the sleeve protocol, in which UDP is used to contact the router on the destination host to transfer packets. Subsequently, those packets are captured by the kernel and passed to the target pod.

One way to install Weave Net on a Kubernetes cluster is to apply a daemonset which will automatically install the necessary containers for running the plugin on each node. Once up and running, all pods will use this network for their communication. The peers are self-configuring, so you can add more nodes to the cluster and they’ll use the same network without further configuration from your side.

Flannel
Flannel is a networking plugin created by CoreOS. It implements cluster networking in Kubernetes by creating an overlay network. It starts a daemon called flanneld on each node. This daemon runs under a pod whose name starts with kube-flannel-ds-*. When assigning IP addresses, Flannel allocates a small subset of IPs of each host (by default, 10.244.X.0/24). This subset is brought from a larger, preconfigured address space. This subset is used to assign an IP address of each pod on the node.

Flannel uses the Kubernetes API server or the cluster’s etcd database directly to store information about the assigned subnets, network configuration, and the host IP address.

Packet forwarding among hosts is done through several protocols like UDP and VXLAN.

For the specific instructions for installing and running Flannel, please use this document.

TL;DR
In this article, we provided a gentle introduction to Kubernetes networking. The way Kubernetes was designed gives the user much freedom in choosing which components work together best (while still abiding by some governing rules). In the first part of this article, we started by explaining how Kubernetes designers thought about how networking should be implemented within the cluster.

Then we briefly discussed some of the Kubernetes networking concepts that help you understand the variety of networking plugins offered by the community. In the second part, we listed a sample of the most well-known networking plugins that are available for Kubernetes, how each of them was designed, and how they implement networking within the cluster.

Deciding on which network plugin to use with your coming project largely depends on what this project is, its network requirements, and the level and type of network security it needs to implement.

https://www.magalix.com/blog/kubernetes-cluster-networking-101
https://www.magalix.com/blog/kubernetes-cluster-networking-101
https://www.magalix.com/blog/kubernetes-cluster-networking-101

https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/kubernetes-demo
https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/kubernetes-demo


https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

Debugging DNS Resolution
This page provides hints on diagnosing DNS problems.

Before you begin
You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using minikube or you can use one of these Kubernetes playgrounds:

Katacoda
Play with Kubernetes

Your cluster must be configured to use the CoreDNS addon or its precursor, kube-dns.
Your Kubernetes server must be at or later than version v1.6. To check the version, enter kubectl version.

Create a simple Pod to use as a test environment
admin/dns/dnsutils.yaml Copy admin/dns/dnsutils.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: k8s.gcr.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always

  Note: This example creates a pod in the default namespace. DNS name resolution for services depends on the namespace of the pod. For more information, review DNS for Services and Pods.
Use that manifest to create a Pod:

kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
pod/dnsutils created
…and verify its status:

kubectl get pods dnsutils
NAME      READY     STATUS    RESTARTS   AGE
dnsutils   1/1       Running   0          <some-time>

Once that Pod is running, you can exec nslookup in that environment. If you see something like the following, DNS is working correctly.

kubectl exec -i -t dnsutils -- nslookup kubernetes.default
Server:    10.0.0.10
Address 1: 10.0.0.10

Name:      kubernetes.default
Address 1: 10.0.0.1

If the nslookup command fails, check the following:

Check the local DNS configuration first
Take a look inside the resolv.conf file. (See Customizing DNS Service and Known issues below for more information)

kubectl exec -ti dnsutils -- cat /etc/resolv.conf

https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/


https://www.reddit.com/r/kubernetes/comments/orqwte/pod_access_host_network_without_hostnetworktrue/

https://kubernetes.io/docs/concepts/security/pod-security-policy/#host-namespaces
HostNetwork - Controls whether the pod may use the node network namespace. Doing so gives the pod access to the loopback device, services listening on localhost, and could be used to snoop on network activity of other pods on the same node.
https://stackoverflow.com/questions/64792961/kubernetes-pod-with-hostnetwork-true-cannot-reach-external-ips-of-services-in-th
https://stackoverflow.com/questions/70242041/how-to-connect-to-local-network-from-kubernetes-pod




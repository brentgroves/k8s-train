Network traffic Path
MetalLB load balancer IP range
Ingress controller service devoted to load balancing.
1 of the 3 Ingress Nginx primary or secondary ingress controller pods 


https://learnk8s.io/kubernetes-network-packets

Kubernetes networking requirements
Before diving into the details on how packets flow inside a Kubernetes cluster, let's first clear up the requirements for a Kubernetes network.

The Kubernetes networking model defines a set of fundamental rules:

A pod in the cluster should be able to freely communicate with any other pod without the use of Network Address Translation (NAT).
Any program running on a cluster node should communicate with any pod on the same node without using NAT.
Each pod has its own IP address (IP-per-Pod), and every other pod can reach it at that same address.
Those requirements don't restrict the implementation to a single solution.

Instead, they describe the properties of the cluster network in general terms.

In satisfying those constraints, you will have to solve the following challenges:

How do you make sure that containers in the same pod behave as if they are on the same host?
Can the pod reach other pods in the cluster?
Can the pod reach services? And are the services load balancing requests?
Can the pod receive traffic external to the cluster?
In this article, you will focus on the first three points, starting with intra-pod networking or container-to-container communication.


How Linux network namespaces work in a pod
Let's consider a main container hosting the application and another running alongside it.

In this example, you have a pod with an Nginx container and another with busybox:

pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: container-1
      image: busybox
      command: ['/bin/sh', '-c', 'sleep 1d']
    - name: container-2
      image: nginx
When deployed, the following things happen:

The pod gets its own network namespace on the node.
An IP address is assigned to the pod, and the ports are shared between the two containers.
Both containers share the same networking namespace and can see each other on localhost.
The network configuration happens lightning fast in the background.

However, let's take a step back and try to understand why the above is needed for containers to run.

In Linux, the network namespaces are separate, isolated, logical spaces.

You can think of network namespaces as taking the physical network interface and slicing it into smaller independent parts.

Each part can be configured separately and with its own networking rules and resources.

Those can range from firewall rules, interfaces (virtual or physical), routes, and everything else networking-related.

Those can range from firewall rules, interfaces (virtual or physical), routes, and everything else networking-related.


The physical network interface holds the root network namespace.
1/2
The physical network interface holds the root network namespace.

NEXT 

You can use Linux network namespaces to create isolated networks. Each network is independent and doesn't talk to the others unless you configure it to.


The physical interface has to process all the real packets in the end, so all virtual interfaces are created from that.

The network namespaces can be managed by the ip-netns management tool, and you can use ip netns list to list the namespaces on a host.

Please note that when a network namespace is created, it will be present under /var/run/netns but Docker doesn't always respect that.

For example, these are namespaces from a Kubernetes node:

The physical interface has to process all the real packets in the end, so all virtual interfaces are created from that.

The network namespaces can be managed by the ip-netns management tool, and you can use ip netns list to list the namespaces on a host.

Please note that when a network namespace is created, it will be present under /var/run/netns but Docker doesn't always respect that.

For example, these are namespaces from a Kubernetes node:

bash
ip netns list
cni-0f226515-e28b-df13-9f16-dd79456825ac (id: 3)
cni-4e4dfaac-89a6-2034-6098-dd8b2ee51dcd (id: 4)
cni-7e94f0cc-9ee8-6a46-178a-55c73ce58f2e (id: 2)
cni-7619c818-5b66-5d45-91c1-1c516f559291 (id: 1)
cni-3004ec2c-9ac2-2928-b556-82c7fb37a4d8 (id: 0)
Notice the cni- prefix; this means that the namespace creation has been taken care of by a CNI.

CNI (Container Network Interface), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers, along with a number of plugins.

When you create a pod, and that pod gets assigned to a node, the CNI will:

Assign an IP address.
Attach the container(s) to the network.
If the pod contains multiple containers like above, both containers are put in the same namespace.


When you create a pod, first the container runtime creates a network namespace for the containers.
1/3
When you create a pod, first the container runtime creates a network namespace for the containers.

NEXT 

Then, the CNI takes lead and assigns it an IP address.

So what happens when you list the containers on a node?

You can SSH into a Kubernetes node and explore the namespaces:

So what happens when you list the containers on a node?

You can SSH into a Kubernetes node and explore the namespaces:

bash
lsns -t net
        NS TYPE NPROCS   PID USER     NETNSID NSFS                           COMMAND
4026531992 net     171     1 root  unassigned /run/docker/netns/default      /sbin/init noembed norestore
4026532286 net       2  4808 65535          0 /run/docker/netns/56c020051c3b /pause
4026532414 net       5  5489 65535          1 /run/docker/netns/7db647b9b187 /pause
Where lsns is a command for listing all available namespaces on a host.

Keep in mind that there are multiple namespace types in Linux.

Where is the Nginx container?

What are those pause containers?

The pause container creates the network namespace in the pod
Let's list all the processes on the node and check if we can find the Nginx container:

bash
lsns
        NS TYPE   NPROCS   PID USER            COMMAND
# truncated output
4026532414 net         5  5489 65535           /pause
4026532513 mnt         1  5599 root            sleep 1d
4026532514 uts         1  5599 root            sleep 1d
4026532515 pid         1  5599 root            sleep 1d
4026532516 mnt         3  5777 root            nginx: master process nginx -g daemon off;
4026532517 uts         3  5777 root            nginx: master process nginx -g daemon off;
4026532518 pid         3  5777 root            nginx: master process nginx -g daemon off;
The container is listed in the mount (mnt), Unix time-sharing (uts) and PID (pid) namespace, but not in the networking namespace (net).

Unfortunately, lsns only shows the lowest PID for each process, but you can further filter based on the process ID.

You can retrieve all namespaces for the Nginx container with:

The container is listed in the mount (mnt), Unix time-sharing (uts) and PID (pid) namespace, but not in the networking namespace (net).

Unfortunately, lsns only shows the lowest PID for each process, but you can further filter based on the process ID.

You can retrieve all namespaces for the Nginx container with:

bash
sudo lsns -p 5777
       NS TYPE   NPROCS   PID USER  COMMAND
4026531835 cgroup    178     1 root  /sbin/init noembed norestore
4026531837 user      178     1 root  /sbin/init noembed norestore
4026532411 ipc         5  5489 65535 /pause
4026532414 net         5  5489 65535 /pause
4026532516 mnt         3  5777 root  nginx: master process nginx -g daemon off;
4026532517 uts         3  5777 root  nginx: master process nginx -g daemon off;
4026532518 pid         3  5777 root  nginx: master process nginx -g daemon off;
The pause process again, and this time it's holding the network namespace hostage.

What is that?

Every pod in the cluster has an additional hidden container running in the background called pause.

If you list the containers running on a node and grab the pause containers:

bash
docker ps | grep pause
fa9666c1d9c6   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_kube-dns-599484b884-sv2js…
44218e010aeb   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_blackbox-exporter-55c457d…
5fb4b5942c66   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_kube-dns-599484b884-cq99x…
8007db79dcf2   k8s.gcr.io/pause:3.4.1  "/pause"  k8s_POD_konnectivity-agent-84f87c…
You will see that for each assigned pod on the node, a pause container is automatically paired with it.

This pause container is responsible for creating and holding the network namespace.

Creating the namespace?

Yes and no.

The network namespace creation is done by the underlaying container runtime. Usually containerd or CRI-O.

Just before the pod is deployed and container created, (among other things) it's the runtime responsibility to create the network namespace.

Instead of running ip netns and creating the network namespace manually, the container runtime does this automatically.

Back to the pause container.

It contains very little code and instantly goes to sleep as soon as deployed.

However, it is essential and plays a crucial role in the Kubernetes ecosystem.


However, it is essential and plays a crucial role in the Kubernetes ecosystem.


When you create a pod, the container runtime creates a network namespace with a sleep container.
1/3
When you create a pod, the container runtime creates a network namespace with a sleep container.

NEXT 


How can a container that goes to sleep be useful?

To understand its utility, let's imagine having a pod with two containers like in the previous example, but no pause container.

As soon as the container starts, the CNI:

Makes the busybox container join the previous network namespace.
Assigns an IP address.
Attaches the containers to the network.
What happens if the Nginx crashes?

The CNI will have to go through all of the steps again and the network will be disrupted for both containers.

Since it's unlikely that a sleep container can have any bug, it's usually a safer and more robust choice to create the network namespace.

If one of the containers inside the pod crashes, the remaining can still reply to any network requests.

The Pod is assigned a single IP address
I mentioned that the pod and both containers receive the same IP.

How is that configured?

Inside the pod network namespace, an interface is created, and an IP address is assigned.

Let's verify that.

First, find the pod's IP address:

bash
kubectl get pod multi-container-pod -o jsonpath={.status.podIP}
10.244.4.40
Next, let's find the relevant network namespace.

Since network namespaces are created from a physical interface, you will have to access the cluster node.

10.1.210.65 

If you are running minikube, you can try minikube ssh to access the node. If you are running in a cloud provider, there should be some way to access the node over SSH.

Once you are in, let's find the latest named network namespace that was created:

bash
ls -lt /var/run/netns
total 0
-r--r--r-- 1 root root 0 Sep 25 13:34 cni-0f226515-e28b-df13-9f16-dd79456825ac
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-4e4dfaac-89a6-2034-6098-dd8b2ee51dcd
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-7e94f0cc-9ee8-6a46-178a-55c73ce58f2e
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-7619c818-5b66-5d45-91c1-1c516f559291
-r--r--r-- 1 root root 0 Sep 24 09:39 cni-3004ec2c-9ac2-2928-b556-82c7fb37a4d8
In this case it is cni-0f226515-e28b-df13-9f16-dd79456825ac.
https://learnk8s.io/kubernetes-network-packets


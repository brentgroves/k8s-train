https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

The primary reason that Pods can have multiple containers is to support helper applications that assist a primary application. Typical examples of helper applications are data pullers, data pushers, and proxies. Helper and primary applications often need to communicate with each other. Typically this is done through a shared filesystem, as shown in this exercise, or through the loopback network interface, localhost. An example of this pattern is a web server along with a helper program that polls a Git repository for new updates.

The Volume in this exercise provides a way for Containers to communicate during the life of the Pod. If the Pod is deleted and recreated, any data stored in the shared Volume is lost.

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/
Building an application from modular containers means thinking about symbiotic groups of containers that cooperate to provide a service, not one container per service.  In Kubernetes, the embodiment of this modular container service is a Pod.  A Pod is a group of containers that share resources like file systems, kernel namespaces and an IP address.  The Pod is the atomic unit of scheduling in a Kubernetes cluster, precisely because the symbiotic nature of the containers in the Pod require that they be co-scheduled onto the same machine, and the only way to reliably achieve this is by making container groups atomic scheduling units.

Example #1: Sidecar containers
Sidecar containers extend and enhance the "main" container, they take existing containers and make them better.  As an example, consider a container that runs the Nginx web server.  Add a different container that syncs the file system with a git repository, share the file system between the containers and you have built Git push-to-deploy.  But you’ve done it in a modular manner where the git synchronizer can be built by a different team, and can be reused across many different web servers (Apache, Python, Tomcat, etc).  Because of this modularity, you only have to write and test your git synchronizer once and reuse it across numerous apps. And if someone else writes it, you don’t even need to do that.
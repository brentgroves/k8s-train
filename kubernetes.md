Main node is avi-ubu
It's IP range:
172.20.88.16 
172.20.88.17
172.20.88.18
172.20.88.19
https://www.how2shout.com/linux/assign-multiple-ip-address-to-ubuntu-20-04-single-network-interface-gui/ 
use the gui for permanent changes 
use the command line for temporary changes
ip addr add 172.20.88.17/22 brd 172.20.91.255 dev ens3
ip addr del 172.20.88.17/22 brd 172.20.91.255 dev ens3
-- the restart command did not seem to do anything
sudo systemctl restart NetworkManager



https://kubernetes.io/docs/concepts/overview/components/

https://venturebeat.com/2021/12/20/the-state-of-cloud-native-development-kubernetes-is-on-the-rise/

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

https://microk8s.io/docs/services-and-ports
Upon deployment MicroK8s creates a Certificate Authority, a signed server certificate and a service account key file. These files are stored under /var/snap/microk8s/current/certs/. Kubelet and the API server are aware of the same CA and so the signed server certificate is used by the API server to authenticate with kubelet (--kubelet-client-certificate).
printenv $KUBECONFIG
Certificate Authority
kubectl cluster-info

Packet Flow:  

Ingress - bare-metal-ingress 

Service
An abstract way to expose an application running on a set of Pods as a network service.
With Kubernetes you don't need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.
Software multitenancy is a software architecture in which a single instance of software runs on a server and serves multiple tenants. Systems designed in such manner are "shared". A tenant is a group of users who share a common access with specific privileges to the software instance.


Pods 

https://kubernetes.io/docs/concepts/services-networking/ingress/ 

Controllers
In robotics and automation, a control loop is a non-terminating loop that regulates the state of a system.

Here is one example of a control loop: a thermostat in a room.

When you set the temperature, that's telling the thermostat about your desired state. The actual room temperature is the current state. The thermostat acts to bring the current state closer to the desired state, by turning equipment on or off.

In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

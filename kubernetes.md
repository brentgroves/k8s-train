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
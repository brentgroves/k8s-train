MicroK8s is a low-ops, minimal production Kubernetes.

MicroK8s is an open-source system for automating deployment, scaling, and management of containerised applications. It provides the functionality of core Kubernetes components, in a small footprint, scalable from a single node to a high-availability production cluster.

https://discuss.kubernetes.io/t/microk8s-failed-to-join-rpi-cluster-error-code-500/14767/6
https://microk8s.io/docs/addon-dashboard
Remember to append microk8s to all commands
microk8s.kubectl config view --raw > ~/.kube/config
https://kubernetes.io/docs/concepts/overview/components/

https://microk8s.io/docs/services-and-ports
Upon deployment MicroK8s creates a Certificate Authority, a signed server certificate and a service account key file. These files are stored under /var/snap/microk8s/current/certs/. Kubelet and the API server are aware of the same CA and so the signed server certificate is used by the API server to authenticate with kubelet (--kubelet-client-certificate).
printenv $KUBECONFIG
Certificate Authority
kubectl cluster-info

https://microk8s.io/docs/clustering
https://microk8s.io/docs/clustering

From the node you wish to join to this cluster, run the following:
microk8s join 172.20.88.16:25000/032155ae4b27cb33b18aa74e64a60616/98962d6f2ce8

If the node you are adding is not reachable through the default interface you can use one of the following:
 microk8s join 172.20.88.16:25000/032155ae4b27cb33b18aa74e64a60616/98962d6f2ce8
 microk8s join 172.18.0.1:25000/032155ae4b27cb33b18aa74e64a60616/98962d6f2ce8
 microk8s join 172.17.0.1:25000/032155ae4b27cb33b18aa74e64a60616/98962d6f2ce8

Avilla main cluster 1 ip configured in metal b
microk8s enable metallb:172.20.88.16:172.20.88.16


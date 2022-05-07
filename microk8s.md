MicroK8s is a low-ops, minimal production Kubernetes.

MicroK8s is an open-source system for automating deployment, scaling, and management of containerised applications. It provides the functionality of core Kubernetes components, in a small footprint, scalable from a single node to a high-availability production cluster.

How to install High Availability Cluster
https://www.youtube.com/watch?v=dNT5uEeJBSw&t=391s

https://microk8s.io/docs/addon-dashboard
Remember to append microk8s to all commands

Issue: could not deploy a helm chart until I did this.
microk8s.kubectl config view --raw > ~/.kube/config
https://kubernetes.io/docs/concepts/overview/components/

Simple deployment of 2 apps
https://ridwanfajar.medium.com/getting-started-with-microk8s-up-and-running-kubernetes-locally-310640dae156

Deploy docker image to cluster
kubectl create deployment microbot --image=dontrebootme/microbot:v1
Microbot require port 80 on the container (pods) to run it’s service. But it could be mapped into any port in host via NodePort
check deployment
microk8s kubectl get all --all-namespaces | grep microbot
This deployment created a replicaset and pod as well as the deployment entry.

exposing that deployment via nodeport service definition
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0
A NodePort service is the most primitive way to get external traffic directly to your service. NodePort, as the name implies, opens a specific port on all the Nodes (the VMs), and any traffic that is sent to this port is forwarded to the service. We can let
Kubernetes pick this port.
microk8s kubectl expose deployment microbot --type=NodePort --port=80 --name=microbot-service
microk8s kubectl get all --all-namespaces | grep microbot
kubectl get svc --all-namespaces


The most advanced solution is ingress which 
This will let you do both path based and subdomain based routing to backend services. For example, you can send everything on foo.yourdomain.com to the foo service, and everything under the yourdomain.com/bar/ path to the bar service.
When would you use this?
Ingress is probably the most powerful way to expose your services, but can also be the most complicated. There are many types of Ingress controllers, from the Google Cloud Load Balancer, Nginx, Contour, Istio, and more. There are also plugins for Ingress controllers, like the cert-manager, that can automatically provision SSL certificates for your services.

Ingress is the most useful if you want to expose multiple services under the same IP address, and these services all use the same L7 protocol (typically HTTP). You only pay for one load balancer if you are using the native GCP integration, and because Ingress is “smart” you can get a lot of features out of the box (like SSL, Auth, Routing, etc)


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

https://ridwanfajar.medium.com/getting-started-with-microk8s-up-and-running-kubernetes-locally-310640dae156
Local Registry
D. Deploy My Own Docker Image
It might have a different way to start with. You have to build your image first in your local machine then we have to push it to MicroK8S internal registry. Afterwards, we could deploy our own local image to MicroK8S.

D.1. Register my local Docker image to MicroK8S
Before we push our image to MicroK8S, we have to save built image as a Tarball. Once you have built your image using docker build , you are able to save the image as Tarball using docker save command. The Tarball will be pushed to MicroK8S and will be reusable as long you specify “never pull” policy on the deployment configuration.

Now you may push the image to MicroK8S by execute those commands below:

$ sudo docker save pokemon-api > pokemon-api.tar
$ microk8s ctr images ls | grep pokemon
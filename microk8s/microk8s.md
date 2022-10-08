# remove microk8s
microk8s stop
sudo snap remove microk8s

https://microk8s.io/#install-microk8s
sudo snap install microk8s --classic --channel=1.21
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube

I believe this is in the dotfiles
alias kubectl='microk8s kubectl'

# Add nodes to cluster
https://microk8s.io/docs/clustering

microk8s add-node
microk8s join 172.20.88.16:25000/552672885f4fcf007e153eb1ee425c2d/6bf7e3972626
kubectl get no
microk8s status

Verify nodes have been added
kubectl get node -o wide
All nodes are shown as master nodes with microk8s status.

# Enable the necessary MicroK8s Add ons: 
microk8s enable dns 
microk8s enable helm3

This addon adds an NGINX Ingress Controller for MicroK8s. It is enabled by running the command:
https://microk8s.io/docs/addon-ingress
# enables primary NGINX ingress controller
microk8s enable ingress
# wait for microk8s to be ready, ingress now enabled
microk8s status --wait-ready | head -n9


# Configure Load Balancer and Ingress Controlers.
https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/
The ingress microk8s add-on provides a convenient way to setup a primary NGINX ingress controller.

Setting up a MetalLB/Ingress service
For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress to properly balance across the cluster ( make sure you have also enabled ingress in MicroK8s first, with microk8s enable ingress). To do this, it requires a service. A suitable ingress service is defined here:

The MetalB is lv 4 and the ingress is lv 7 of the osi model
so the traffic is first seen by the metalb loadbalancer which then sends it to one of the ingress controllers through the service you define to decide which pod to 
send it to using an ingress object.

For stand-alone systems:
avi-ubu
microk8s enable metallb:172.20.88.16-172.20.88.16
frt-ubu
microk8s enable metallb:172.20.1.190-172.20.1.190
moto
microk8s enable metallb:10.1.1.83-10.1.1.83

For production clusters:
reports0
microk8s enable metallb:10.1.0.116-10.1.0.116,10.1.0.117-10.1.0.117,10.1.0.118-10.1.0.118
sleep 15
reports1
microk8s enable metallb:10.1.0.110-10.1.0.110,10.1.0.111-10.1.0.111,10.1.0.112-10.1.0.112
sleep 15
reports1
microk8s enable metallb:10.1.0.113-10.1.0.113,10.1.0.114-10.1.0.114,10.1.0.115-10.1.0.115
sleep 15


reports01 = 10.1.0.116
reports02 = 10.1.0.117
reports03 = 10.1.0.118
moto = 10.1.1.83
frt-ubu = 172.20.1.190
avi-ubu = 172.20.88.16








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

# wait for microk8s to be ready, metallb now enabled
microk8s status --wait-ready | head -n16

# view MetalLB objects
kubectl get all -n metallb-system

# show MetalLB configmap with IP used
kubectl get configmap/config -n metallb-system -o yaml

# Enable Secondary Ingress
To create a secondary ingress, we must go beyond using the microk8s ‘ingress’ add-on.  I have put a DaemonSet definition into github as nginx-ingress-secondary-micro8s-controller.yaml, which you can apply like below.

# apply DaemonSet that creates secondary ingress
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/add_secondary_ingress/templates/nginx-ingress-secondary-microk8s-controller.yaml.j2
https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html
Ansible uses Jinja2 templating to enable dynamic expressions and access to variables and facts. You can use templating with the template module.

kubectl apply -f nginx-ingress-secondary-microk8s-controller.yaml
# you should now see both:
# 'nginx-ingress-microk8s-controller' and 
# 'nginx-ingress-private-microk8s-controller'
kubectl get all --namespace ingress

# Create Ingress Services
  ## If multiple ingress services do this:
You need to create two Services, one for the primary ingress using the first MetalLB IP address and another for the secondary using the second MetalLB IP address. I could not choose the exact MetalLB IP address for the service but microK8s choose one for both services.

I just removed the IP address because I could not get this to work if I manually specified the IP addresses.

kubectl apply -f nginx-ingress-service-primary-and-secondary.yaml

## shows 'ingress' and 'ingress-secondary' Services
## both ClusterIP as well as MetalLB IP addresses
kubectl get services --namespace ingress


  ## If only one ingress controller do this:
kubectl apply -f nginx-ingress-service.yaml

## shows 'ingress' service
## both ClusterIP as well as MetalLB IP address
kubectl get services --namespace ingress

# test deployment of ingress
# apply first deployment
kubectl apply -f golang-hello-world-web.yaml

# apply second deployment for secondary ingress controller.
kubectl apply -f golang-hello-world-web2yaml


# show deployment(s) and then pod(s)
kubectl get deployments 
kubectl get pods -o wide
kubectl get services 
These apps are now available at their internal pod IP address.

# internal ip of primary pod
export primaryPodIP=$(microk8s kubectl get pods -l app=golang-hello-world-web -o=jsonpath="{.items[0].status.podIPs[0].ip}")

# internal IP of secondary pod
export secondaryPodIP=$(microk8s kubectl get pods -l app=golang-hello-world-web2 -o=jsonpath="{.items[0].status.podIPs[0].ip}")

# check pod using internal IP
curl http://${primaryPodIP}:8080/myhello/

# check pod using internal IP
curl http://${secondaryPodIP}:8080/myhello2/

With internal pod IP proven out, move up to the IP at the  Service level.

# IP of primary service
export primaryServiceIP=$(microk8s kubectl get service/golang-hello-world-web-service -o=jsonpath="{.spec.clusterIP}")

# IP of secondary service
export secondaryServiceIP=$(microk8s kubectl get service/golang-hello-world-web-service2 -o=jsonpath="{.spec.clusterIP}")

# check primary service
curl http://${primaryServiceIP}:8080/myhello/

# check secondary service
curl http://${secondaryServiceIP}:8080/myhello2/

These validations proved out the pod and service independent of the NGINX ingress controller.  Notice all these were using insecure HTTP on port 8080, because the Ingress controller step in the following step is where TLS is layered on.

# generate addition certificates from reports01:
To generate additional certificates for the root certificate
stored on reports01 at 10.1.0.116 follow the instruction in the 
certificates directory of the
git@github.com:brentgroves/linux-utils.git repository.

# Install the certificates onto the cluster:
cd ~/src
git clone git@github.com:brentgroves/linux-utils.git
cd linux-utils/certificates

kubectl create -n default secret tls tls-credential --key=reports01-key.pem --cert=reports01.pem
kubectl create -n default secret tls tls-secondary-credential --key=reports02-key.pem --cert=reports02.pem
kubectl create -n default secret tls tls-credential --key=reports11-key.pem --cert=reports11.pem
kubectl create -n default secret tls tls-secondary-credential --key=reports12-key.pem --cert=reports12.pem
kubectl create -n default secret tls tls-credential --key=reports22-key.pem --cert=reports22.pem
kubectl create -n default secret tls tls-secondary-credential --key=reports23-key.pem --cert=reports23.pem
kubectl create -n default secret tls tls-credential --key=moto-key.pem --cert=moto.pem
kubectl create -n default secret tls tls-credential --key=avi-ubu-key.pem --cert=avi-ubu.pem
kubectl create -n default secret tls tls-secondary-credential --key=frt-ubu-key.pem --cert=frt-ubu.pem















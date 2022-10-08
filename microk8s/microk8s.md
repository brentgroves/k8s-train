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

Enable the necessary MicroK8s Add ons: 
microk8s enable dns 
microk8s enable helm3

This addon adds an NGINX Ingress Controller for MicroK8s. It is enabled by running the command:
https://microk8s.io/docs/addon-ingress
# enables primary NGINX ingress controller
microk8s enable ingress
# wait for microk8s to be ready, ingress now enabled
microk8s status --wait-ready | head -n9





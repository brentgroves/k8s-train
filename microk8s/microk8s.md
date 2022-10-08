# remove microk8s
microk8s stop
sudo snap remove microk8s

https://microk8s.io/#install-microk8s
sudo snap install microk8s --classic --channel=1.21
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube

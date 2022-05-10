sudo snap remove microk8s

sudo apt install open-iscsi
open-iscsi is already the newest version (2.0.874-7.1ubuntu6.2).
Once the package is installed you will find the following files:

I did not change these 2 files.
/etc/iscsi/iscsid.conf
/etc/iscsi/initiatorname.iscsi

https://microk8s.io/#install-microk8s
sudo snap install microk8s --classic --channel=1.21
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube


microk8s will continue to run until you stop it.
microk8s stop
microk8s start

https://microk8s.io/docs/clustering

microk8s add-node
microk8s join 172.20.88.16:25000/552672885f4fcf007e153eb1ee425c2d/6bf7e3972626
microk8s kubectl get no
microk8s status


Enable the necessary MicroK8s Add ons: 
microk8s enable dashboard dns helm3
If RBAC is not enabled access the dashboard using the default token retrieved with:
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
microk8s kubectl -n kube-system describe secret $token

set up nfs server
https://microk8s.io/docs/nfs
sudo apt-get install nfs-kernel-server
sudo mkdir -p /srv/nfs
sudo chown nobody:nogroup /srv/nfs
sudo chmod 0777 /srv/nfs

Edit the /etc/exports file. Make sure that the IP addresses of all your MicroK8s nodes are able to mount this share. For example, to allow all IP addresses in the 10.0.0.0/24 subnet:
sudo mv /etc/exports /etc/exports.bak
echo '/srv/nfs 10.0.0.0/24(rw,sync,no_subtree_check)' | sudo tee /etc/exports

# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)

# cat /etc/exports
/srv/nfs 172.20.88.0/23(rw,sync,no_subtree_check)
/srv/nfs 10.1.0.0/22(rw,sync,no_subtree_check)
/srv/nfs 172.20.0.0/23(rw,sync,no_subtree_check)

sudo systemctl restart nfs-kernel-server

Enable the Helm3 addon (if not already enabled) and add the repository for the NFS CSI driver:
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update

GitHub - kubernetes-csi/csi-driver-nfs: This driver allows Kubernetes to access NFS server on Linux node. ... Install driver on a Kubernetes cluster

Then, install the Helm chart under the kube-system namespace with:
microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
    --namespace kube-system \
    --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet

 At this point, you should also be able to list the available CSI drivers in your Kubernetes cluster …

microk8s kubectl get csidrivers


microk8s enable metallb:172.20.88.16-172.20.88.19,10.1.1.83,172.20.1.190

kubectl describe pods -n openebs openebs-apiserver-bc6bc5986-xk4rw

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

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-18-04
sudo mkdir -p /nfs/general
sudo mount 172.20.88.16:/srv/nfs /nfs/general

Enable the Helm3 addon (if not already enabled) and add the repository for the NFS CSI driver:
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update

GitHub - kubernetes-csi/csi-driver-nfs: This driver allows Kubernetes to access NFS server on Linux node. ... Install driver on a Kubernetes cluster
Overview. CSI is an open standard API that enables Kubernetes to expose arbitrary storage systems to containerized workloads. Kubernetes volumes are managed by vendor-specific storage drivers, which have historically been compiled into Kubernetes binaries.


Then, install the Helm chart under the kube-system namespace with:
microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet

hmicrok8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
    --namespace kube-system \
    --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet


 At this point, you should also be able to list the available CSI drivers in your Kubernetes cluster …

microk8s kubectl get csidrivers

create a storage class
microk8s kubectl apply -f - < sc-nfs.yaml

Create a new PVC
The final step is to create a new PersistentVolumeClaim using the nfs-csi storage class. This is as simple as specifying storageClassName: nfs-csi in the PVC definition, for example:

# pvc-nfs.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi

microk8s kubectl get sc 
Then create the PVC with:

microk8s kubectl apply -f - < pvc-nfs.yaml

If everything has been configured correctly, you should be able to check the PVC…
microk8s kubectl describe pvc my-pvc

What is OCI Object Storage
https://docs.oracle.com/en-us/iaas/Content/Object/Concepts/objectstorageoverview.htm
OCI Object Storage provides a dedicated (non-shared) storage 'namespace' or container unique to each customer for all stored buckets and objects. This encapsulation provides end-to-end visibility and reduces the risk of exposed buckets.

https://microk8s.io/docs/addon-metallb
microk8s enable metallb:172.20.88.16-172.20.88.19
update its configmap

https://metallb.universe.tf/configuration/
https://github.com/metallb/metallb/issues/308#:~:text=To%20migrate%20an%20IP%20address,within%20the%20metallb%2Dsystem%20namespace.
How to update the IP address range
look at the old metalb config map
kubectl -n metallb-system get cm config
kubectl get configmap config -n metallb-system -o yaml
# note the old IPs allocated to the services
kubectl get svc --all-namespaces
k8s-namespace-frt   proxy-public           LoadBalancer   10.152.183.155   172.20.88.16   80:30622/TCP             47h
# delete the old configmap
kubectl -n metallb-system delete cm config
# apply the new configmap
kubectl apply -f metalb.yaml

https://microk8s.io/docs/addon-metallb
Setting up a MetalLB/Ingress service
For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress to properly balance across the cluster ( make sure you have also enabled ingress in MicroK8s first, with microk8s enable ingress). To do this, it requires a service. A suitable ingress service is defined here:

microk8s enable ingress


microk8s enable openebs

sudo mkdir -p /srv/k8s
sudo chown nobody:nogroup /srv/k8s
sudo chmod 0777 /srv/k8s


kubectl get pods -n openebs
kubectl describe pods -n openebs openebs-apiserver-bc6bc5986-cmzbg


microk8s.kubectl apply -f local-storage-dir.yaml
kubectl get sc --all-namespaces

microk8s.kubectl apply -f local-storage-dir.yaml

create the persistant volume claim
kubectl apply -f local-storage-dir-pvc.yaml

Look at the PersistentVolumeClaim:
kubectl get pvc local-storage-dir-pvc

The output shows that the STATUS is Pending. This means PVC has not yet been used by an application pod. The next step is to create a Pod that uses your PersistentVolumeClaim as a volume.

NAME                 STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS       AGE
local-hostpath-pvc   Pending                                      openebs-hostpath   3m7s


kubectl apply -f local-storage-dir-pod.yaml

Look at the PersistentVolumeClaim:
kubectl get pvc local-storage-dir-pvc

kubectl exec hello-local-storage-dir-pod -- cat /mnt/store/greet.txt

https://openebs.io/docs/user-guides/localpv-hostpath#create-storageclass
kubectl delete pod hello-local-storage-dir-pod
kubectl delete pvc local-storage-dir-pvc


Remember to append microk8s to all commands
microk8s.kubectl config view --raw > ~/.kube/config

helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

helm upgrade --cleanup-on-fail \
  --install helm-release-frt jupyterhub/jupyterhub \
  --namespace k8s-namespace-frt \
  --create-namespace \
  --version=1.2.0 \
  --values config.yaml

how to change
modify config.yaml
then run 

helm upgrade --cleanup-on-fail \
    helm-release-frt jupyterhub/jupyterhub \
  --namespace k8s-namespace-frt \
  --version=1.2.0 \
  --values config.yaml

Thank you for installing JupyterHub!
Your release is named "helm-release-frt" and installed into the namespace "k8s-namespace-frt".
You can check whether the hub and proxy are ready by running:
 kubectl --namespace=k8s-namespace-frt get pod

and watching for both those pods to be in status 'Running'.

You can find the public (load-balancer) IP of JupyterHub by running:

  kubectl -n k8s-namespace-frt get svc proxy-public -o jsonpath='{.status.loadBalancer.ingress[].ip}'

It might take a few minutes for it to appear!

To get full information about the JupyterHub proxy service run:

  kubectl --namespace=k8s-namespace-frt get svc proxy-public
#to remain sane set namespace default
microk8s.kubectl config set-context $(microk8s.kubectl config current-context) --namespace k8s-namespace-frt

If you have questions, please:

  1. Read the guide at https://z2jh.jupyter.org
  2. Ask for help or chat to us on https://discourse.jupyter.org/
  3. If you find a bug please report it at https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues




OPEN ISSUE
https://kubernetes.slack.com
OPENEBS IS CRASHING



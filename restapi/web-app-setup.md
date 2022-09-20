# how to deploy the hotel web application container in app.py
#1st build the docker image
docker build -t brentgroves/app1:1 .
docker push brentgroves/app1:1
#
 deploy web-application
kubectl apply -f web-application-deployment.yaml

# show both deployments and then pods
kubectl get deployments 
kubectl get pods -o wide
kubectl get services 

These apps are now available at their internal pod IP address.

# internal ip of primary pod
export primaryPodIP=$(microk8s kubectl get pods -l app=web-application -o=jsonpath="{.items[0].status.podIPs[0].ip}")

# check pod using internal IP
# add hotel
curl -X POST http://${primaryPodIP}:5000/hotel -H 'Content-Type: application/json' -d '{"id":"1","name":"name1","state":"state1","rooms":"1"}'
curl -X POST http://${primaryPodIP}:5000/hotel -H 'Content-Type: application/json' -d '{"id":"2","name":"name2","state":"state2","rooms":"2"}'
# list hotels
curl http://${primaryPodIP}:5000/hotel


With internal pod IP proven out, move up to the IP at the  Service level.

# IP of primary service
export primaryServiceIP=$(microk8s kubectl get service/web-application-service -o=jsonpath="{.spec.clusterIP}")

# check service
curl http://${primaryServiceIP}:5000/hotel


These validations proved out the pod and service independent of the NGINX ingress controller.  Notice all these were using insecure HTTP on port 5000, because the Ingress controller step in the following step is where TLS is layered on.

Create TLS key and certificate

Before we expose these services via Ingress, we must create the TLS keys and certificates that will be used when serving traffic.

# delete secret 
kubectl delete secret tls-credential
kubectl delete secret tls-secondary-credential

# #############################################
# Start of mkcert method of creating certificates
# ################################################
https://github.com/FiloSottile/mkcert
My preferred way of making certs.
using mkcert you can add multiple domain names to the certificate, SAN certificate, but I only have one domain specified 
# shows 'ingress' and 'ingress-secondary' Services
# both ClusterIP as well as MetalLB IP addresses
kubectl get services --namespace ingress
# verify the correct host for the MetalLB IP service address
# since I can't specify in ip address for the ingress controller services
# always verify the external ip address of each ingress controller service. 
sudo nvim /etc/hosts
reports.k8s
10.1.0.116      reports01 # primary ingress
10.1.0.117      reports02 # secondary ingress
10.1.0.118      reports03
reports-dev.k8s
10.1.0.110      reports11 # primary ingress
10.1.0.111      reports12 # secondary ingress
10.1.0.112      reports13
10.1.0.113      reports14 # not used yet
tooling.k8s
10.1.1.83       moto
172.20.88.16    avi-ubu # primary ingress service
172.20.1.190    frt-ubu # secondary ingress service

go to the certificates directory of the 
git clone git@github.com:brentgroves/linux-utils.git
repository to create the tls secrets
linux-utils will have these commands to deploy secrets to the 3 k8s clusters:
kubectl create -n default secret tls tls-credential --key=reports01-key.pem --cert=reports01.pem
kubectl create -n default secret tls tls-secondary-credential --key=reports02-key.pem --cert=reports02.pem
kubectl create -n default secret tls tls-credential --key=reports11-key.pem --cert=reports11.pem
kubectl create -n default secret tls tls-secondary-credential --key=reports12-key.pem --cert=reports12.pem
kubectl create -n default secret tls tls-credential --key=avi-ubu-key.pem --cert=avi-ubu.pem
kubectl create -n default secret tls tls-secondary-credential --key=frt-ubu-key.pem --cert=frt-ubu.pem

# shows both tls secrets
kubectl get secrets --namespace default
kubectl describe secret tls-credential
kubectl describe secret tls-secondary-credential

Deploy via Ingress
Thank you Father, to make these services available to the outside world, we need to expose them via the NGINX Ingress and MetalLB addresses.
NGINX = engineX
# create primary ingress

verify the correct host name are set in yaml

kubectl apply -f web-application-ingress.yaml

# verify ingress objects point to web-application pod
# and have the same host defined in the tls certificate
# get the pod ip from this command:
kubectl describe ingress 

Validate URL endpoints
The Ingress requires that the proper FQDN headers be sent by your browser, so it is not sufficient to do a GET against the MetalLB IP addresses.  You have two options:

add the FQDN, such as ‘microk8s.local’ and ‘microk8s-secondary.local’ entries to your local /etc/hosts file
OR use the curl ‘–resolve’ flag to specify the FQDN to IP mapping which will send the host header correctly
Here is an example of pulling from the primary and secondary Ingress using entries in the /etc/hosts file.

# check  ingress
choose 1 of the following:
curl -k https://avi-ubu/hotel
curl -k https://reports01/hotel
curl https://reports11/hello
For windows do this:
curl https://reports01/myhello/ --ssl-no-revoke 
For Ubuntu do this:
curl https://reports01/myhello/ --cacert /usr/local/share/ca-certificates/mkcert_development_CA_303095335489122417061412993970225104069.crt 


apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress -- I BELIEVE THIS MUST BE SET TO INGRESS
spec:
  selector:
    name: nginx-ingress-microk8s
  type: LoadBalancer
  # loadBalancerIP is optional. MetalLB will automatically allocate an IP 
  # from its pool if not specified. You can also specify one manually.
  # I could not get this service to run when I manually specified an IP address.
  # loadBalancerIP: 172.20.88.16
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443

(base)  ✘ bgroves@avi-ubu  ~  kubectl get services --all-namespaces
NAMESPACE     NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
default       kubernetes   ClusterIP      10.152.183.1     <none>         443/TCP                      50m
kube-system   kube-dns     ClusterIP      10.152.183.10    <none>         53/UDP,53/TCP,9153/TCP       44m
ingress       ingress      LoadBalancer   10.152.183.207   172.20.88.16   80:30022/TCP,443:32183/TCP   35s

Now there is a load-balancer which listens on an arbitrary IP and directs traffic towards one of the listening ingress controllers.
kubectl get all --namespace ingress

in robotics and automation, a control loop is a non-terminating loop that regulates the state of a system.
Here is one example of a control loop: a thermostat in a room.
When you set the temperature, that's telling the thermostat about your desired state. The actual room temperature is the current state. The thermostat acts to bring the current state closer to the desired state, by turning equipment on or off.
In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.

With the Ingress addon enabled, a HTTP/HTTPS ingress rule can be created with an Ingress resource. For example:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: http-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: some-service
          servicePort: 80

Test if ingress 
Test ingress
# get definition of first service/deployment
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web.yaml.j2

# apply first one
microk8s kubectl apply -f golang-hello-world-web.yaml.j2

microk8s kubectl get deployments
microk8s kubectl get pods
get clusterip
microk8s kubectl get services
21
11 + 8 in plant 2
2
1
2

# internal ip of primary pod
kubectl get pods -l app=golang-hello-world-web -o=jsonpath="{.items[0].status.podIPs[0].ip}"
export primaryPodIP=$(microk8s kubectl get pods -l app=golang-hello-world-web -o=jsonpath="{.items[0].status.podIPs[0].ip}")

curl http://${primaryPodIP}:8080/myhello/
curl http://10.1.210.68:8080/myhello/

With internal pod IP proven out, move up to the IP at the  Service level.

# IP of primary service
export primaryServiceIP=$(microk8s kubectl get service/golang-hello-world-web-service -o=jsonpath="{.spec.clusterIP}")

# check primary service
curl http://${primaryServiceIP}:8080/myhello/

# These validations proved out the pod and service independent of the NGINX ingress controller.  Notice all these were using insecure HTTP on port 8080, because the Ingress controller step in the following step is where TLS is layered on.

# Create TLS key and certificate
# Before we expose these services via Ingress, we must create the TLS keys and certificates that will be used when serving traffic.

# Primary ingress will use TLS with CN=microk8s.local
# Secondary ingress will use TLS with CN=microk8s-secondary.local
# The best way to do this is with either a commercial certificate, or creating your own custom CA and SAN certificates.  But this article is  striving for simplicity, so we will simply generate self-signed certificates using a simple script I wrote.

# download and change script to executable
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/cert-with-ca/files/microk8s-self-signed.sh


  # create self-signed cert
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
-keyout microk8s.local.key -out microk8s.local.crt \
-subj "/C=US/ST=CA/L=SFO/O=myorg/CN=$FQDN"

https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/

# download and change script to executable
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/cert-with-ca/files/microk8s-self-signed.sh


  # create self-signed cert
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
-keyout microk8s.local.key -out microk8s.local.crt \
-subj "/C=US/ST=CA/L=SFO/O=myorg/CN=$FQDN"

  # create pem which contains key and cert
sudo cat microk8s.local.crt microk8s.local.key | sudo tee -a microk8s.local.pem

  # smoke test, view CN
openssl x509 -noout -subject -in microk8s.local.crt

sudo chown $USER /tmp/microk8s.local.{pem,crt,key}

# create primary tls secret for 'microk8s.local'
microk8s kubectl create -n default secret tls tls-credential --key=/tmp/microk8s.local.key --cert=/tmp/microk8s.local.crt

https://kubernetes.io/docs/concepts/configuration/secret/
# shows both tls secrets
microk8s kubectl get secrets --namespace default

Deploy via Ingress
Finally, to make these services available to the outside world, we need to expose them via the NGINX Ingress and MetalLB addresses.

# create primary ingress
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web-on-nginx.yaml.j2

microk8s kubectl apply -f golang-hello-world-web-on-nginx.yaml.j2



curl http://172.20.88.16 -I
curl http://172.20.1.190 -I
curl http://10.1.1.83 -I



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



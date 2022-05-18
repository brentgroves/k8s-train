https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/
# check ClusterIP and port of first and second service
microk8s kubectl get services

# internal ip of primary pod
primaryPodIP=$(microk8s kubectl get pods -l app=golang-hello-world-web -o=jsonpath="{.items[0].status.podIPs[0].ip}")

# internal IP of secondary pod
secondaryPodIP=$(microk8s kubectl get pods -l app=golang-hello-world-web2 -o=jsonpath="{.items[0].status.podIPs[0].ip}")

# check pod using internal IP
curl http://${primaryPodIP}:8080/myhello/

# check pod using internal IP
curl http://${secondaryPodIP}:8080/myhello2/

With internal pod IP proven out, move up to the IP at the  Service level.

# IP of primary service
primaryServiceIP=$(microk8s kubectl get service/golang-hello-world-web-service -o=jsonpath="{.spec.clusterIP}")

# IP of secondary service
secondaryServiceIP=$(microk8s kubectl get service/golang-hello-world-web-service2 -o=jsonpath="{.spec.clusterIP}")

# check primary service
curl http://${primaryServiceIP}:8080/myhello/

# check secondary service
curl http://${secondaryServiceIP}:8080/myhello2/

Create TLS key and certificate
Before we expose these services via Ingress, we must create the TLS keys and certificates that will be used when serving traffic.

Primary ingress will use TLS with CN=microk8s.local
Secondary ingress will use TLS with CN=microk8s-secondary.local
The best way to do this is with either a commercial certificate, or creating your own custom CA and SAN certificates.  But this article is striving for simplicity, so we will simply generate self-signed certificates using a simple script I wrote.

# download and change script to executable
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/cert-with-ca/files/microk8s-self-signed.sh

chmod +x microk8s-self-signed.sh

# run openssl commands that generate our key + certs in /tmp
./microk8s-self-signed.sh

# change permissions so they can be read by normal user
sudo chmod go+r /tmp/*.{key,crt}

# show key and certs created
ls -l /tmp/microk8s*


# create primary tls secret for 'microk8s.local'
microk8s kubectl create -n default secret tls tls-credential --key=/tmp/microk8s.local.key --cert=/tmp/microk8s.local.crt

# create secondary tls secret for 'microk8s-secondary.local'
microk8s kubectl create -n default secret tls tls-secondary-credential --key=/tmp/microk8s-secondary.local.key --cert=/tmp/microk8s-secondary.local.crt

# shows both tls secrets
microk8s kubectl get secrets --namespace default


Deploy via Ingress
Finally, to make these services available to the outside world, we need to expose them via the NGINX Ingress and MetalLB addresses.

# create primary ingress
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web-on-nginx.yaml.j2

microk8s kubectl apply -f golang-hello-world-web-on-nginx.yaml.j2

# create secondary ingress 
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web-on-nginx2.yaml.j2 

microk8s kubectl apply -f golang-hello-world-web-on-nginx2.yaml.j2

# show primary and secondary Ingress objects
# primary available at 'microk8s.local'
# secondary available at 'microk8s-secondary.local'
microk8s kubectl get ingress --namespace default

# shows primary and secondary ingress objects tied to MetalLB IP
microk8s kubectl get services --namespace ingress


Validate URL endpoints
The Ingress requires that the proper FQDN headers be sent by your browser, so it is not sufficient to do a GET against the MetalLB IP addresses.  You have two options:

add the ‘microk8s.local’ and ‘microk8s-secondary.local’ entries to your local /etc/hosts file
OR use the curl ‘–resolve’ flag to specify the FQDN to IP mapping which will send the host header correctly
Here is an example of pulling from the primary and secondary Ingress using entries in the /etc/hosts file.

# validate you have entries to 192.168.1.141 and .142
grep microk8s /etc/hosts

# check primary ingress
curl -k https://microk8s.local/myhello/

# check secondary ingress
curl -k https://microk8s-secondary.local/myhello2/

https://microk8s.io/docs/addon-ingress
https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/

https://kubernetes.github.io/ingress-nginx/examples/tls-termination/

Setting up a MetalLB/Ingress service
For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress to properly balance across the cluster ( make sure you have also enabled ingress in MicroK8s first, with microk8s enable ingress). To do this, it requires a service. A suitable ingress service is defined here:

The MetalB is lv 4 and the ingress is lv 7 of the osi model
so the traffic is first seen by the metalb loadbalance which then sends it to one of the ingress controllers to decide which pod to 
send it to using an ingress object.
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


Test ingress
# get definition of first service/deployment
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web.yaml.j2

# apply first one
microk8s kubectl apply -f golang-hello-world-web.yaml.j2


Configure networking.
https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/
The ingress microk8s add-on provides a convenient way to setup a primary NGINX ingress controller.

Setting up a MetalLB/Ingress service
For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress to properly balance across the cluster ( make sure you have also enabled ingress in MicroK8s first, with microk8s enable ingress). To do this, it requires a service. A suitable ingress service is defined here:

The MetalB is lv 4 and the ingress is lv 7 of the osi model
so the traffic is first seen by the metalb loadbalance which then sends it to one of the ingress controllers to decide which pod to 
send it to using an ingress object.

DO THIS FIRST BEFORE ENABLING METALB
This only has to be done on one node. I ran it on the master node.
This addon adds an NGINX Ingress Controller for MicroK8s. It is enabled by running the command:
microk8s enable ingress

Now you can enable metalb
microk8s enable metallb:172.20.88.16-172.20.88.16,172.20.1.190-172.20.1.190,10.1.1.83-10.1.1.83

Setting up a MetalLB/Ingress service
For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress to properly balance across the cluster ( make sure you have also enabled ingress in MicroK8s first, with microk8s enable ingress). To do this, it requires a service. A suitable ingress service is defined here:
microk8s kubectl apply -f ingress-service.yaml

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



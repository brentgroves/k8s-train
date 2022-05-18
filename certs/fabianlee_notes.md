https://medium.com/avmconsulting-blog/how-to-secure-applications-on-kubernetes-ssl-tls-certificates-8f7f5751d788
https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/


# view NGINX ingress objects created by 'ingress' add-on
kubectl get all --namespace ingress

# apply DaemonSet that creates secondary ingress
https://docs.nginx.com/nginx-ingress-controller/installation/running-multiple-ingress-controllers/
https://www.bmc.com/blogs/kubernetes-daemonset/


wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/add_secondary_ingress/templates/nginx-ingress-secondary-microk8s-controller.yaml.j2


Create Ingress Services
Then you need to create two Services, one for the primary ingress using the first MetalLB IP address and another for the secondary using the second MetalLB IP address.

Download the nginx-ingress-service-primary-and-secondary.yaml.j2 template, and do a couple of replacements before applying with kubectl.

# download template
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/add_secondary_nginx_ingress/templates/nginx-ingress-service-primary-and-secondary.yaml.j2



# download and change script to executable
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/cert-with-ca/files/microk8s-self-signed.sh


  # create self-signed cert
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
-keyout microk8s.local.key -out microk8s.local.crt \
-subj "/C=US/ST=CA/L=SFO/O=myorg/CN=microk8s.local"

  # create self-signed cert
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
-keyout microk8s-secondary.local.key -out microk8s-secondary.local.crt \
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
microk8s kubectl describe secrets/tls-credential


Deploy via Ingress
Finally, to make these services available to the outside world, we need to expose them via the NGINX Ingress and MetalLB addresses.

# create primary ingress
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web-on-nginx.yaml.j2


https://github.com/FiloSottile/mkcert
This is my preferred way of making certs.

# #############################################
# Start of mkcert method of creating certificates
# ################################################
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

https://geekflare.com/openssl-commands-certificates/
# there are different ways to do the same thing the avm blog uses
# different parameters in openssl to achieve the same results
# the above link shows different openssl in a tasks list.

https://microk8s.io/docs/addon-ingress
https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/

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

          https://fabianlee.org/2021/07/29/kubernetes-microk8s-with-multiple-metallb-endpoints-and-nginx-ingress-controllers/

Test ingress
# get definition of first service/deployment
wget https://raw.githubusercontent.com/fabianlee/microk8s-nginx-istio/main/roles/golang-hello-world-web/templates/golang-hello-world-web.yaml.j2

# apply first one
microk8s kubectl apply -f golang-hello-world-web.yaml.j2
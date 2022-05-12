https://discuss.kubernetes.io/t/addon-metallb/11790
https://microk8s.io/docs/addon-metallb

Setting up a MetalLB/Ingress service
For load balancing in a MicroK8s cluster, MetalLB can make use of Ingress to properly balance across the cluster ( make sure you have also enabled ingress in MicroK8s first, with microk8s enable ingress). To do this, it requires a service. A suitable ingress service is defined here:

The MetalB is lv 4 and the ingress is lv 7 of the osi model
so the traffic is first seen by the metalb loadbalance which then sends it to one of the ingress controllers to decide which pod to 
send it to using an ingress object.

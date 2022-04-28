# k8s-train
https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/
kubectl describe deployment rss-site
kubectl delete deploy rss-site

https://microk8s.io/docs/services-and-ports
Upon deployment MicroK8s creates a Certificate Authority, a signed server certificate and a service account key file. These files are stored under /var/snap/microk8s/current/certs/. Kubelet and the API server are aware of the same CA and so the signed server certificate is used by the API server to authenticate with kubelet (--kubelet-client-certificate).
printenv $KUBECONFIG
Certificate Authority
kubectl cluster-info

https://microk8s.io/docs/clustering
From the node you wish to join to this cluster, run the following:
microk8s join 172.20.1.190:25000/79aea7091d6795599b912f8ec434c8c7/b8a7ad089308

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 172.20.1.190:25000/79aea7091d6795599b912f8ec434c8c7/b8a7ad089308 --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 172.20.1.190:25000/79aea7091d6795599b912f8ec434c8c7/b8a7ad089308
microk8s join 172.17.0.1:25000/79aea7091d6795599b912f8ec434c8c7/b8a7ad089308
(b


https://www.freecodecamp.org/news/what-is-a-helm-chart-tutorial-for-kubernetes-beginners/
helm upgrade --install --create-namespace rss-site ./rss-chart/chart \
  --set image.tag=v1.0.0 \
  --set env=production \
  --set environment.SENDGRID_APIKEY=myKey \
  --set environment.DEFAULT_FROM_ADDRESS="my@email.com" \
  --set environment.DEFAULT_FROM_NAME="Lucas Santos"
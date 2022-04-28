https://helm.sh/docs/intro/using_helm/
https://v3-1-0.helm.sh/

https://www.containiq.com/post/helm-charts
https://www.tutorialworks.com/helm-cheatsheet/
helm install myapp-instance ./path/to/chart


helm repo add bitnami https://charts.bitnami.com/bitnami
https://artifacthub.io/packages/helm/artifact-hub/artifact-hub
helm repo add artifact-hub https://artifacthub.github.io/helm-charts

helm search repo nginx
helm install nginx bitnami/nginx
kubectl get all
kubectl get svc --namespace default -w nginx
helm list

helm install demo1 ./demo

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

NAME: nginx
LAST DEPLOYED: Wed Apr 27 18:20:48 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 10.1.4
APP VERSION: 1.21.6

** Please be patient while the chart is being deployed **

NGINX can be accessed through the following DNS name from within your cluster:

    nginx.default.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w nginx'

    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services nginx)
    export SERVICE_IP=$(kubectl get svc --namespace default nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"

https://k8s.io/examples/application/shell-demo.yaml

kubectl create -f https://k8s.io/examples/application/shell-demo.yaml

kubectl exec -it shell-demo -- /bin/bash

apt-get update
apt-get install curl
curl external-web

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs/
kubectl get node -o wide
kubectl apply --validate -f deployment.yaml
kubectl get pods -o wide
kubectl get deployments -o wide
kubectl get all --namespace ingress
kubectl get services --all-namespaces
kubectl get secret db-user-pass -o jsonpath='{.data.password2}' | base64 --decode

kubectl get deployment etl-pod -o jsonpath={.kind}
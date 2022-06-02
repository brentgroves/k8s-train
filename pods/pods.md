# create pod
kubectl apply -f etl-pod.yaml

# verify pod was created 
kubectl get pods -o wide

# delete pod
kubectl delete pod etl-pod
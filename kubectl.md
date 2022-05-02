https://kubernetes.io/docs/reference/kubectl/cheatsheet/
kubectl get deployment --namespace default


https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/
kubectl describe deployment rss-site
kubectl delete deploy rss-site

kubectl delete deploy --namespace default nginx


Create Deployment 
Manifest: https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/ 
Kubernetes, remember, manages container-based applications and services. In the case of a K8s Deployment, you’re creating a set of resources to be managed. For example, where we previously created a single instance of the Pod, we might create a Kubernetes Deployment YAML example to tell Kubernetes to manage a set of replicas of that Pod — literally, a ReplicaSet — to make sure that a certain number of them are always available.
Blueprint for creating the pod. 

kubectl create deployment nginx-deply --image=nginx 

kubectl create -f deployment.yaml



Kubectl get deployment --all-namespaces

kubectl get pod 
# List all pods in ps output format with more information (such as node name and internal IP addresses)
kubectl get pods -o wide


kubectl get replicaset – manages the replicas of the pod 

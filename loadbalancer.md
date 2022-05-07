https://platform9.com/blog/using-metallb-to-add-the-loadbalancer-service-to-kubernetes-environments/
https://docs.k0sproject.io/head/examples/metallb-loadbalancer/
https://platform9.com/blog/using-metallb-to-add-the-loadbalancer-service-to-kubernetes-environments/
disadvantage of nodeport is that they expose a high numbered port in a range set 30000-?
NodePort
What about NodePort? It’s definitely something we could set up and manage, however the key word here would be “manage” – which is something we are trying to get away from. What would this setup look like? We would end up exposing deployments with a service type of NodePort which would give us the Nodes as Endpoints with high numbered ports, something like 192.168.16.22:30000. For each node where we have this socket, we would need to manage a LoadBalancer outside of Kubernetes to distribute traffic between them. If we are using a static list then we need to be able to poll/update the list when there are changes. Sounds like additional work that could result in dropped traffic if the list hasn’t been updated.



apiVersion: v1
kind: Service
metadata:
  name: nginx
  annotations:
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl create -f lb-deployment.yaml

kubectl get service nginx

curl http://192.168.86.10
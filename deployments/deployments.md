https://blog.container-solutions.com/kubernetes-deployment-strategies

Recreate - best for development environment
A deployment defined with a strategy of type Recreate will terminate all the running instances then recreate them with the newer version.

 
spec:
  replicas: 3
  strategy:
    type: Recreate
	
  
kubectl apply --validate -f deployment.yaml

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
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

In this example:

A Deployment named nginx-deployment is created, indicated by the .metadata.name field.

The Deployment creates three replicated Pods, indicated by the .spec.replicas field.

The .spec.selector field defines how the Deployment finds which Pods to manage. In this case, you select a label that is defined in the Pod template (app: nginx). However, more sophisticated selection rules are possible, as long as the Pod template itself satisfies the rule.

Note: The .spec.selector.matchLabels field is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". All of the requirements, from both matchLabels and matchExpressions, must be satisfied in order to match.
The template field contains the following sub-fields:

The Pods are labeled app: nginxusing the .metadata.labels field.
The Pod template's specification, or .template.spec field, indicates that the Pods run one container, nginx, which runs the nginx Docker Hub image at version 1.14.2.
Create one container and name it nginx using the .spec.template.spec.containers[0].name field.


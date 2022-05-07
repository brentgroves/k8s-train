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
      app: nginx -- label in the pod template to manage 
  template:  -- this is the pod template
    metadata:
      labels:
        app: nginx  -- this is a label in the pod template that 
        -- the deployment will manage
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

The .spec.selector field defines how the Deployment finds which Pods to manage. In this case, you select a label that is defined in the Pod template (app: nginx). However, more sophisticated selection rules are possible, as long as the Pod template itself satisfies the rule.

The template field contains the following sub-fields:
The Pods are labeled app: nginx using the .metadata.labels field.
The Pod template's specification, or .template.spec field, indicates that the Pods run one container, nginx, which runs the nginx Docker Hub image at version 1.14.2.
Create one container and name it nginx using the .spec.template.spec.containers[0].name field.

kubectl delete deployment deployment_name
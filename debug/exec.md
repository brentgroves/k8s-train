https://k8s.io/examples/application/shell-demo.yaml

kubectl create -f https://k8s.io/examples/application/shell-demo.yaml

kubectl exec --stdin --tty shell-demo -- /bin/bash

kubectl exec -it shell-demo -- /bin/bash
kubectl exec -it shell-demo -- /bin/bash

apt-get update
apt-get install curl
curl external-web


https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/


Getting a shell to a container 
In this exercise, you create a Pod that has one container. The container runs the nginx image. Here is the configuration file for the Pod:

application/shell-demo.yaml Copy application/shell-demo.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: shell-demo
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  hostNetwork: true
  dnsPolicy: Default
Create the Pod:

kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml
Verify that the container is running:

kubectl get pod shell-demo
Get a shell to the running container:


kubectl exec --stdin --tty shell-demo -- /bin/bash
kubectl exec --stdin --tty py-etl-training -- /bin/bash

Note: The double dash (--) separates the arguments you want to pass to the command from the kubectl arguments.

Running individual commands in a container
In an ordinary command window, not your shell, list the environment variables in the running container:

kubectl exec shell-demo env
Experiment with running other commands. Here are some examples:

kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts

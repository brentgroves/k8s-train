
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
Assigning pods to nodes
kubernetes.io/hostname=reports3

Add a label to a node
List the nodes in your cluster, along with their labels:

kubectl get nodes --show-labels
The output is similar to this:

NAME      STATUS    ROLES    AGE     VERSION        LABELS
worker0   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker0
worker1   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker1
worker2   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker2

Choose one of your nodes, and add a label to it:

kubectl label nodes <your-node-name> disktype=ssd

kubectl label nodes reports3 mysql=yes

Verify that your chosen node has a disktype=ssd label:

kubectl get nodes --show-labels

Create a pod that gets scheduled to your chosen node
This pod configuration file describes a pod that has a node selector, disktype: ssd. This means that the pod will get scheduled on a node that has a disktype=ssd label.

pods/pod-nginx.yaml Copy pods/pod-nginx.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd

    Use the configuration file to create a pod that will get scheduled on your chosen node:

kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml
Verify that the pod is running on your chosen node:

kubectl get pods --output=wide
The output is similar to this:

NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0

Create a pod that gets scheduled to specific node
You can also schedule a pod to one specific node via setting nodeName.

pods/pod-nginx-specific-node.yaml Copy pods/pod-nginx-specific-node.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
Use the configuration file to create a pod that will get scheduled on foo-node only.


https://k8s.networkop.co.uk/services/nodeport/
NodePort builds on top of the ClusterIP Service and provides a way to expose a group of Pods to the outside world. At the API level, the only difference from the ClusterIP is the mandatory service type which has to be set to NodePort, the rest of the values can remain the same.

Whenever a new Kubernetes cluster gets built, one of the available configuration parameters is service-node-port-range which defines a range of ports to use for NodePort allocation and usually defaults to 30000-32767. One interesting thing about NodePort allocation is that it is not managed by a controller. The configured port range value eventually gets passed to the kube-apiserver as an argument and allocation happens as the API server saves a Service resource into its persistent storage (e.g. etcd cluster); a unique port is allocated for both Nodeport and LoadBalancer services. So by the time the Service definition makes it to the persistent storage, it already contains a couple of extra fields:



Moxa
Since all nodes would map to the same service we could configure all moxa from a plant to one node to lesson the bottleneck.

 

https://ridwanfajar.medium.com/getting-started-with-microk8s-up-and-running-kubernetes-locally-310640dae156
$ microk8s kubectl create deployment microbot --image=dontrebootme/microbot:v1
$ microk8s kubectl expose deployment microbot --type=NodePort --port=80 --name=microbot-service

Once you have ran those commands above. You may check the deployment using microk8s kubectl get all --all-namespaces | grep microbot . Then, you will find microbot-service is mapped to port 31895 . Then, you may open Microbot service in web browser via http://localhost:31895.

http://avi-ubu:31895 also works.


Create 2nd deployment:
$ microk8s kubectl create deployment hello-world --image=tutum/hello-world:latest
$ microk8s kubectl expose deployment hello-world --type=NodePort --port=80 --name=hello-world-service

Once you have ran those commands above. You may check the deployment using microk8s kubectl get all --all-namespaces | grep hello-world . Then, you will find hello-world-service is mapped to port 31197 . Then, you may open Microbot service in web browser via http://localhost:31197.

Limitation:
I think all the nodes expose this port.
Both for the section C.1 and C.2 might have different port mapping for every service exposure attempt. So you need to check them through microk8s kubectl get -all --all-namespaces

apiVersion: v1
kind: Service
metadata:
  labels:
    app: FE
  name: FE
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: FE
  type: NodePort


  kubectl apply -f lb-deployment.yaml
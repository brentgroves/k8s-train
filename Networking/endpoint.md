https://theithollow.com/2019/02/04/kubernetes-endpoints/

Kubernetes – Endpoints
February 4, 2019 8  By ERIC SHANKS
It’s quite possible that you could have a Kubernetes cluster but never have to know what an endpoint is or does, even though you’re using them behind the scenes. Just in case you need to use one though, or if you need to do some troubleshooting, we’ll cover the basics of Kubernetes endpoints in this post.

Endpoints – The Theory
During the post where we first learned about Kubernetes Services, we saw that we could use labels to match a frontend service with a backend pod automatically by using a selector. If any new pods had a specific label, the service would know how to send traffic to it. Well the way that the service knows to do this is by adding this mapping to an endpoint. Endpoints track the IP Addresses of the objects the service send traffic to. When a service selector matches a pod label, that IP Address is added to your endpoints and if this is all you’re doing, you don’t really need to know much about endpoints. However, you can have Services where the endpoint is a server outside of your cluster or in a different namespace (which we haven’t covered yet).

What you should know about endpoints is that there is a list of addresses your services will send traffic and its managed through endpoints. Those endpoints can be updated automatically through labels and selectors, or you can manually configure your endpoints depending on your use case.

Endpoints – In Action
Let’s take a look at some endpoints that we’ve used in our previous manifests. Lets deploy this manifest as we did in our previous posts and take a look to see what our endpoints are doing.

apiVersion: apps/v1 #version of the API to use
kind: Deployment #What kind of object we're deploying
metadata: #information about our object we're deploying
  name: nginx-deployment #Name of the deployment
  labels: #A tag on the deployments created
    app: nginx
spec: #specifications for our object
  replicas: 2 #The number of pods that should always be running
  selector: #which pods the replica set should be responsible for
    matchLabels:
      app: nginx #any pods with labels matching this I'm responsible for.
  template: #The pod template that gets deployed
    metadata:
      labels: #A tag on the replica sets created
        app: nginx
    spec:
      containers:
      - name: nginx-container #the name of the container within the pod
        image: nginx #which container image should be pulled
        ports:
        - containerPort: 80 #the port of the container within the pod
---
apiVersion: v1 #version of the API to use
kind: Service #What kind of object we're deploying
metadata: #information about our object we're deploying
  name: ingress-nginx #Name of the service
spec: #specifications for our object
  type: NodePort #Ignore for now discussed in a future post
  ports: #Ignore for now discussed in a future post
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30001 
    protocol: TCP
  selector: #Label selector used to identify pods
    app: nginx


  We can deploy this manifest running:

kubectl apply -f [manifest file].yml
Code language: CSS (css)
Now that its done we should be able to see the endpoints that were automatically created when our service selector matched our pod label. To see this we can query our endpoints through kubectl.

kubectl get endpoints

NAME                              ENDPOINTS                                               AGE
golang-hello-world-web-service    10.1.113.132:8080                                       7d3h
mysql                             10.1.113.134:3306                                       2d2h
golang-hello-world-web-service2   10.1.113.136:8080                                       4d20h
udpclient-svc                     10.1.113.137:5005,10.1.113.137:5006                     2d2h
kubernetes                        10.1.1.83:16443,172.20.1.190:16443,172.20.88.16:16443   7d5h
(base)  bgroves@avi-ubu 

Now those endpoints should be the IP addresses of our pods that we deployed in our manifest. To test this, lets use the get pods command with the -o wide switch to show more output.

kubectl get pods -o wide

NAME                                       READY   STATUS    RESTARTS   AGE     IP             NODE                     NOMINATED NODE   READINESS GATES
golang-hello-world-web-5c545766bf-xc89q    1/1     Running   0          4d23h   10.1.113.132   frt-ubu.busche-cnc.com   <none>           <none>
mysql-7cd567cc69-rbftx                     1/1     Running   0          46h     10.1.113.134   frt-ubu.busche-cnc.com   <none>           <none>
golang-hello-world-web2-6cd5bcb745-xkxxh   1/1     Running   0          91m     10.1.113.136   frt-ubu.busche-cnc.com   <none>           <none>
udpclient-f758d96dc-lr24b                  1/1     Running   0          91m     10.1.113.137   frt-ubu.busche-cnc.com   <none>           <none>

You can see that the IP addresses associated with the pods matches the endpoints. So we proved that the endpoints are matching under the hood.

How about if we want to manually edit our endpoints if we don’t have a selector? Maybe we’re trying to have a resource to access an external service that doesn’t live within our Kubernetes cluster? We could create our own endpoint to do this for us. A great example might be an external database service for our web or app containers.

Let’s look at an example where we’re using an endpoint to access an external resource from a container. In this case we’ll access a really simple web page just for a test. For reference, I accessed this service from my laptop first to prove that it’s working. If you’re doing this in your lab, you’ll need to spin up a web server and modify the IP Addresses accordingly.

I know that it’s not very exciting, but it’ll get the job done. Next up we’ll deploy our Endpoint and a service with no selector. The following manifest should do the trick. Notice the ports used and the IP Address specified in the endpoint.

kind: "Service"
  apiVersion: "v1"
  metadata:
    name: "external-web"
  spec:
    ports:
      -
        name: "apache"
        protocol: "TCP"
        port: 80
        targetPort: 80 
---
  kind: "Endpoints"
  apiVersion: "v1"
  metadata:
    name: "external-web" 
  subsets: 
    -
      addresses:
        -
          ip: "10.10.50.53" #The IP Address of the external web server
      ports:
        -
          port: 80 
          name: "apache"

Once the service and endpoint are deployed, we’ll deploy a quick container into our cluster so we can use it to curl our web page as a test. We’ll use the imperative commands instead of a manifest file this time.

NAME                              ENDPOINTS                                               AGE
mysql                             10.1.113.134:3306                                       2d3h
udpclient-svc                     10.1.113.137:5005,10.1.113.137:5006                     2d3h
golang-hello-world-web-service    10.1.113.132:8080                                       7d4h
golang-hello-world-web-service2   10.1.113.136:8080                                       4d22h
kubernetes                        10.1.1.83:16443,172.20.1.190:16443,172.20.88.16:16443   7d6h
busche-sql                        10.1.2.74:1433                                          14s


https://kubernetes.io/docs/tutorials/
kubectl create -f https://k8s.io/examples/application/shell-demo.yaml

NAME                                       READY   STATUS    RESTARTS   AGE     IP             NODE                     NOMINATED NODE   READINESS GATES
mysql-7cd567cc69-rbftx                     1/1     Running   0          47h     10.1.113.134   frt-ubu.busche-cnc.com   <none>           <none>
udpclient-f758d96dc-lr24b                  1/1     Running   0          176m    10.1.113.137   frt-ubu.busche-cnc.com   <none>           <none>
golang-hello-world-web-5c545766bf-xc89q    1/1     Running   0          5d      10.1.113.132   frt-ubu.busche-cnc.com   <none>           <none>
golang-hello-world-web2-6cd5bcb745-xkxxh   1/1     Running   0          176m    10.1.113.136   frt-ubu.busche-cnc.com   <none>           <none>
shell-demo                                 1/1     Running   0          6m58s   10.1.1.83      moto.busche-cnc.com      <none>           <none>

kubectl exec -it shell-demo -- /bin/bash

apt-get update
apt-get install curl
curl external-web



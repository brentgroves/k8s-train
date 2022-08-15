docker build --tag brentgroves/api1:1
https://blog.devgenius.io/how-to-deploy-rest-api-application-using-mysql-on-the-kubernetes-cluster-4c806de1a48
# https://medium.com/bb-tutorials-and-thoughts/how-to-dockerize-the-python-rest-api-with-flask-library-d2b51dd4a0ae

// run the container
docker run -d -p 5000:5000 --name api1 brentgroves/api1:1

docker push brentgroves/api1:1

// list the container
docker ps
// logs
docker logs python-restapi
// exec into running container
docker exec -it api1 /bin/sh

Use config-map.yaml to define values of environment variables that will be used in the API application

You should add the correct value for MYSQL_ROOT_HOST
Use the output of the following command

kubectl apply -f config-map.yaml

To deploy an application use web-application-deployment.yaml Add your image-name:image-tag — value that was used in the docker build command

To create deployment
kubectl apply -f web-application-deployment.yaml

To verify creation
kubectl get deployment
kubectl delete deployment web-application

To create service for this deployment run
The pod ip address may change because it is determined by the deployment, but the service IP address will stay the same. And 
the service controller scans for all the pods with the selector condition and chooses one to pass traffic to.
Added service definition to deployment so the following create service is not needed.
kubectl create service clusterip web-application --tcp=80:5000
kubectl get services
kubectl delete service web-application

These apps are now available at their internal pod IP address.
kubectl get pods -o wide
or 
# internal ip of primary pod
export primaryPodIP=$(kubectl get pods -l app=web-application -o=jsonpath="{.items[0].status.podIPs[0].ip}")

To verify all setup run
1st use the pod ip address in the curl command.
# check pod using internal IP
curl http://${primaryPodIP}:5000/hotel -H 'Content-Type: application/json' -d '{"id":"1","name":"name1","state":"state1","rooms":"1"}'
curl -X POST http://10.1.210.65:5000/hotel -H 'Content-Type: application/json' -d '{"id":"1","name":"name1","state":"state1","rooms":"1"}'
curl -X POST http://10.1.210.65:5000/hotel -H 'Content-Type: application/json' -d '{"id":"2","name":"name2","state":"state1","rooms":"2"}'
curl http://10.1.210.65:5000/hotel

kubectl get services
# IP of primary service
export primaryServiceIP=$(kubectl get service/web-application-service -o=jsonpath="{.spec.clusterIP}")
# check primary service
curl http://${primaryServiceIP}:5000/hotel/

Next use the service ip address in the curl command except with the port mapping 80.
curl -X POST http://10.152.183.234:5000/hotel -H 'Content-Type: application/json' -d '{"id":"3","name":"name3","state":"state3","rooms":"3"}'
curl http://10.152.183.234:5000/hotel


These validations proved out the pod and service independent of the NGINX ingress controller.  Notice all these were using insecure HTTP on port 8080, because the Ingress controller step in the following step is where TLS is layered on.





Trouble-shooting:
kubectl exec --stdin --tty web-application-6f5578748f-zl67w  -- /bin/bash
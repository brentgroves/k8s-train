If you don't know the exact path then jsonpath="{}" will list all the name/value pairs of the root of the object.
kubectl get service/golang-hello-world-web-service2 -o=jsonpath="{}"
kubectl get pods -l app=golang-hello-world-web2 -o=jsonpath="{}"
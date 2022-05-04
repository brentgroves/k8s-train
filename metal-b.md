Zero-to-Kubernetes how I installed metallb: microk8s enable metallb:172.20.88.16-172.20.88.16
https://metallb.universe.tf/configuration/
https://github.com/metallb/metallb/issues/308#:~:text=To%20migrate%20an%20IP%20address,within%20the%20metallb%2Dsystem%20namespace.
How to update the IP address range
look at the old metalb config map
kubectl -n metallb-system get cm config
kubectl get configmap config -n metallb-system -o yaml
# note the old IPs allocated to the services
kubectl get svc --all-namespaces
k8s-namespace-frt   proxy-public           LoadBalancer   10.152.183.155   172.20.88.16   80:30622/TCP             47h
# delete the old configmap
kubectl -n metallb-system delete cm config
# apply the new configmap
kubectl apply -f metalb.yaml
# delete the metallb pods
kubectl -n metallb-system delete pod --all
# watch the pods come back up
kubectl -n metallb-system get pods -w

# inspect new IPs of services
kubectl get svc

# test load balance service
kubectl create -f lb-deployment.yaml

$ kubectl get service nginx
NAME    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
nginx   LoadBalancer   10.21.82.141   172.20.88.17   80:30967/TCP   4m40s

curl 172.20.88.17

Could have configured the install more using a values.yaml file like this article.
https://platform9.com/blog/using-metallb-to-add-the-loadbalancer-service-to-kubernetes-environments/


What default configuration happened when the command was ran: microk8s enable metallb:10.0.0.100-10.0.0.200
kubectl get configmap --all-namespaces
kubectl get configmap -n metallb-system
kubectl get configmap config -n metallb-system -o yaml
apiVersion: v1
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.20.88.16-172.20.88.16
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"config":"address-pools:\n- name: default\n  protocol: layer2\n  addresses:\n  - 172.20.88.16-172.20.88.16\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"config","namespace":"metallb-system"}}
  creationTimestamp: "2022-05-02T19:47:27Z"
  name: config
  namespace: metallb-system
  resourceVersion: "8617"
  selfLink: /api/v1/namespaces/metallb-system/configmaps/config
  uid: 730e1612-1e4f-452c-a262-1812c3b246bd

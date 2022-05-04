Zero-to-Kubernetes how I installed metallb: microk8s enable metallb:10.0.0.100-10.0.0.200

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

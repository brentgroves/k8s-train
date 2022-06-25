https://kubernetes.io/docs/concepts/workloads/pods/

dont create pods directly instead use controllers such as depoyments.

https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/

https://stackoverflow.com/questions/40636021/how-to-list-kubernetes-recently-deleted-pods

kubectl logs --previous ${POD_NAME} ${CONTAINER_NAME}
kubectl logs --previous etl-pod etl-pod

Pods and controllers
You can use workload resources to create and manage multiple Pods for you. A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.


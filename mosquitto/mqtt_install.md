git clone https://github.com/halkeye-helm-charts/mosquitto.git

https://techblogs.42gears.com/how-to-deploy-mqtt-broker-to-kubernetes-cluster/
https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/

Create a PersistentVolume referencing a disk in your environment.
Create a MySQL Deployment.
Expose MySQL to other pods in the cluster at a known DNS name.

https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/

Dynamic volume provisioning allows storage volumes to be created on-demand. Without dynamic provisioning, cluster administrators have to manually make calls to their cloud or storage provider to create new storage volumes, and then create PersistentVolume objects to represent them in Kubernetes. The dynamic provisioning feature eliminates the need for cluster administrators to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.




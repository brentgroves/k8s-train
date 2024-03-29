<!-- https://stackoverflow.com/questions/60247100/hostpath-assign-persistentvolume-to-the-specific-work-node-in-cluster -->
Assigning persistent volume to node.
Assigning volume to node
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - ubuntu18-kubeadm-worker1  



https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

Assigning pods to nodes
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



https://stackoverflow.com/questions/49959601/configure-time-zone-to-mysql-docker-container
I created a custome docker file and tested it with docker-compose
to get timezone configured.

https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/
https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/


https://hub.docker.com/_/mysql

https://dev.mysql.com/doc/mysql-installation-excerpt/8.0/en/docker-mysql-more-topics.html
https://betterprogramming.pub/customize-your-mysql-database-in-docker-723ffd59d8fb

clean-up
kubectl delete deployment,svc mysql
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv-volume

Deploy the PV and PVC of the YAML file:
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/
# This assumes that your Node uses "sudo" to run commands
# as the superuser
I don't think you need to make this directory yourself.
sudo mkdir /mnt/data
unless you want to initialize the directory with a file such
index.html but I have never tried not creating it ahead of time.

cd ~/src
git clone git@github.com:brentgroves/k8s-train.git
cd k8s-train
cd <k8s cluster name>
kubectl apply -f mysql-pv.yaml

kubectl describe pv mysql-pv-volume
kubectl describe pvc mysql-pv-claim


Deploy the contents of the YAML file:
kubectl apply -f mysql-deployment_nodeport.yaml


Display information about the Deployment:
kubectl describe deployment mysql

List the pods created by the Deployment:


Inspect the PersistentVolumeClaim:
kubectl describe pvc mysql-pv-claim

kubectl exec --stdin --tty mysql-7cd567cc69-l9c6j -- /bin/bash
mysql -u root -ppassword
mysql -uroot -ppassword
https://stackoverflow.com/questions/49959601/configure-time-zone-to-mysql-docker-container
see if the timezone is correct. use brentgroves/mysql:5.7
select now() 
create database test;
connect using dbeaver to ip and port 31008
Go to k8sdw folder and run test.sql

apt-get install mysql-client
mysql -h 10.1.0.116 -P 31008 -u root -ppassword
mysql -h 10.1.1.83 -P 31008 -u root -ppassword


!!!! error on reports kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
getting reports2 name lookup failure using this command.
I added reports03 to the /etc/hostname table and it worked.
Thank you Father;


https://docs.pivotal.io/tanzu-mysql-kubernetes/1-0/accessing.html


Create a PersistentVolume referencing a disk in your environment.
Create a MySQL Deployment.
Expose MySQL to other pods in the cluster at a known DNS name.

https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/

Dynamic volume provisioning allows storage volumes to be created on-demand. Without dynamic provisioning, cluster administrators have to manually make calls to their cloud or storage provider to create new storage volumes, and then create PersistentVolume objects to represent them in Kubernetes. The dynamic provisioning feature eliminates the need for cluster administrators to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.

The implementation of dynamic volume provisioning is based on the API object StorageClass from the API group storage.k8s.io. A cluster administrator can define as many StorageClass objects as needed, each specifying a volume plugin (aka provisioner) that provisions a volume and the set of parameters to pass to that provisioner when provisioning. A cluster administrator can define and expose multiple flavors of storage (from the same or different storage systems) within a cluster, each with a custom set of parameters. This design also ensures that end users don't have to worry about the complexity and nuances of how storage is provisioned, but still have the ability to select from multiple storage options.

More information on storage classes can be found here.


Enabling Dynamic Provisioning 
To enable dynamic provisioning, a cluster administrator needs to pre-create one or more StorageClass objects for users. StorageClass objects define which provisioner should be used and what parameters should be passed to that provisioner when dynamic provisioning is invoked. The name of a StorageClass object must be a valid DNS subdomain name.

The following manifest creates a storage class "slow" which provisions standard disk-like persistent disks.

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard

The following manifest creates a storage class "fast" which provisions SSD-like persistent disks.

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd

Using Dynamic Provisioning
Users request dynamically provisioned storage by including a storage class in their PersistentVolumeClaim. Before Kubernetes v1.6, this was done via the volume.beta.kubernetes.io/storage-class annotation. However, this annotation is deprecated since v1.9. Users now can and should instead use the storageClassName field of the PersistentVolumeClaim object. The value of this field must match the name of a StorageClass configured by the administrator (see below).

To select the "fast" storage class, for example, a user would create the following PersistentVolumeClaim:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi

This claim results in an SSD-like Persistent Disk being automatically provisioned. When the claim is deleted, the volume is destroyed.


Defaulting Behavior
Dynamic provisioning can be enabled on a cluster such that all claims are dynamically provisioned if no storage class is specified. A cluster administrator can enable this behavior by:

Marking one StorageClass object as default;
Making sure that the DefaultStorageClass admission controller is enabled on the API server.
An administrator can mark a specific StorageClass as default by adding the storageclass.kubernetes.io/is-default-class annotation to it. When a default StorageClass exists in a cluster and a user creates a PersistentVolumeClaim with storageClassName unspecified, the DefaultStorageClass admission controller automatically adds the storageClassName field pointing to the default storage class.

Note that there can be at most one default storage class on a cluster, or a PersistentVolumeClaim without storageClassName explicitly specified cannot be created.

https://kubernetes.io/docs/concepts/storage/
https://kubernetes.io/docs/concepts/storage/persistent-volumes/


Deploy MySQL
You can run a stateful application by creating a Kubernetes Deployment and connecting it to an existing PersistentVolume using a PersistentVolumeClaim. For example, this YAML file describes a Deployment that runs MySQL and references the PersistentVolumeClaim. The file defines a volume mount for /var/lib/mysql, and then creates a PersistentVolumeClaim that looks for a 20G volume. This claim is satisfied by any existing volume that meets the requirements, or by a dynamic provisioner.

Note: The password is defined in the config yaml, and this is insecure. See Kubernetes Secrets for a secure solution.

Run this on each pod
sudo mkdir -p /srv/mysql
sudo chmod 777 /srv/mysql
sudo chown nobody:nogroup /srv/mysql


https://kubernetes.io/docs/concepts/storage/persistent-volumes/
A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany or ReadWriteMany, see AccessModes).

While PersistentVolumeClaims allow a user to consume abstract storage resources, it is common that users need PersistentVolumes with varying properties, such as performance, for different problems. Cluster administrators need to be able to offer a variety of PersistentVolumes that differ in more ways than size and access modes, without exposing users to the details of how those volumes are implemented. For these needs, there is the StorageClass resource.

Binding 
A user creates, or in the case of dynamic provisioning, has already created, a PersistentVolumeClaim with a specific amount of storage requested and with certain access modes. A control loop in the master watches for new PVCs, finds a matching PV (if possible), and binds them together. If a PV was dynamically provisioned for a new PVC, the loop will always bind that PV to the PVC. Otherwise, the user will always get at least what they asked for, but the volume may be in excess of what was requested. Once bound, PersistentVolumeClaim binds are exclusive, regardless of how they were bound. A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.

Claims will remain unbound indefinitely if a matching volume does not exist. Claims will be bound as matching volumes become available. For example, a cluster provisioned with many 50Gi PVs would not match a PVC requesting 100Gi. The PVC can be bound when a 100Gi PV is added to the cluster.


apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/srv/mysql"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi


https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/

kubectl describe deployment mysql

List the pods created by the Deployment:

 kubectl get pods -l app=mysql
The output is similar to this:

 NAME                   READY     STATUS    RESTARTS   AGE
 mysql-63082529-2z3ki   1/1       Running   0          3m
Inspect the PersistentVolumeClaim:

 kubectl describe pvc mysql-pv-claim

 kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword

 Updating 
The image or any other part of the Deployment can be updated as usual with the kubectl apply command. Here are some precautions that are specific to stateful apps:

Don't scale the app. This setup is for single-instance apps only. The underlying PersistentVolume can only be mounted to one Pod. For clustered stateful apps, see the StatefulSet documentation.
Use strategy: type: Recreate in the Deployment configuration YAML file. This instructs Kubernetes to not use rolling updates. Rolling updates will not work, as you cannot have more than one Pod running at a time. The Recreate strategy will stop the first pod before creating a new one with the updated configuration.
Deleting a deployment 
Delete the deployed objects by name:

kubectl delete deployment,svc mysql
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv-volume

If you manually provisioned a PersistentVolume, you also need to manually delete it, as well as release the underlying resource. If you used a dynamic provisioner, it automatically deletes the PersistentVolume when it sees that you deleted the PersistentVolumeClaim. Some dynamic provisioners (such as those for EBS and PD) also release the underlying resource upon deleting the PersistentVolume.

https://kubernetes.io/docs/concepts/storage/volume-snapshots/
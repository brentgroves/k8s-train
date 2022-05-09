https://github.com/openebs/openebs/issues/3317
https://openebs.io/docs/user-guides/quickstart

https://www.youtube.com/watch?v=ay0eSiVi8e8
https://www.youtube.com/watch?v=ay0eSiVi8e8
https://microk8s.io/docs/addon-openebs
OpenEBS installs a set of StorageClass resources but does not mark any of them default. Choose a directory on your host where you want to store data from your cluster. The path can be on the system disk or a separate disk. Create a YAML file called local-storage-dir.yaml with the following contents:
If the BasePath does not exist on the node, OpenEBS Dynamic Local PV Provisioner will attempt to create the directory, when the first Local Volume is scheduled on to that node. You MUST ensure that the value provided for BasePath is a valid absolute path.
https://openebs.io/docs/user-guides/localpv-hostpath#create-storageclass

Create a PersistentVolumeClaim#
The next step is to create a PersistentVolumeClaim. Pods will use PersistentVolumeClaims to request Hostpath Local PV from OpenEBS Dynamic Local PV provisioner.

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-hostpath-pvc
spec:
  storageClassName: local-storage-dir
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G

create the persistant volume claim
kubectl apply -f local-storage-dir-pvc.yaml

Look at the PersistentVolumeClaim:
kubectl get pvc local-storage-dir-pvc

The output shows that the STATUS is Pending. This means PVC has not yet been used by an application pod. The next step is to create a Pod that uses your PersistentVolumeClaim as a volume.

NAME                 STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS       AGE
local-hostpath-pvc   Pending                                      openebs-hostpath   3m7s

https://openebs.io/docs/user-guides/localpv-hostpath#create-storageclass

systemctl enable iscsid
microk8s enable rbac dns openebs storage

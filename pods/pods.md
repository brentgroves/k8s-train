https://kubernetes.io/docs/concepts/workloads/pods/

Pods and controllers
You can use workload resources to create and manage multiple Pods for you. A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.



https://kubernetes.io/docs/tasks/configure-pod-container/

https://www.containiq.com/post/kubernetes-image-pull-policy
imagePullPolicy

Container Registry
A registry is an images repository for development, testing, and access control.  With a container registry, software teams can perform security and vulnerability analysis on container images, improve functionality then deploy them to Kubernetes clusters. To use an image in a cluster, its name and registry are specified in the POD or Deployment configuration file. If a registry hostname is not provided, Kubernetes assumes the Default Docker Registry. 

Different versions of the same image may be needed for different use cases. Tags are added to image names to identify various versions and help isolate the different usage environments. If a tag is not specified in the manifest file, Kubernetes will automatically use the image tagged latest. 

POD Configuration
Administrators may prefer to use the secret for one POD instead of an entire service account. In this case the secret is specified in the pod spec file. The spec file will look something like this:


---
apiVersion: v1
kind: Pod
metadata:
 name: darwin
spec:
 containers:
 - name: app-container1
  image: eu.gcr.io/darwin-imagepull-project/app-container1
  imagePullPolicy: Always
 imagePullSecrets:
 - name: darwin-secret

 Image Pull Policy Options
When creating the POD, one can specify the imagePullPolicyspecification, which guides the Kubelet service on how to pull the specified image during an update. In the above example, it has been set to Always, which means Kubernetes should always pull the image from the registry when updating the container. 

There are three image policy pull options for Kubernetes. 

If imagePullPolicy is set to Always, Kubernetes will always pull the image from the Repository. 
With IfNotPresent, Kubernetes will only pull the image when it does not already exist on the node. 
While with imagePullPolicy set to Never, Kubernetes will never pull the image. 
In case the specification is not stated on the manifest file, Kubernetes will set the policy depending on the image’s tag. If the image is tagged latest, then Kubernetes will assume the imagePullPolicyto be Always. An image with no tag is assumed to be latest, and so its policy is set to Always. Otherwise, the orchestrator will default the imagePullPolicy to IfNotPresent. 

Closing Thoughts
This article has been a high-level guide for image management in Kubernetes. Some concepts studied include container registry and deploying a Docker container using images from a repository. The article also covers Image Pull Policies in-depth, including the various options available. The Kubernetes Image Pull Policy is crucial for running both public images hosted on Docker, Private Repositories and clusters running on legacy images. 

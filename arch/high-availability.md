https://microk8s.io/docs/high-availability
A highly available Kubernetes cluster is a cluster that can withstand a failure on any one of its components and continue serving workloads without interruption. There are three components necessary for a highly available Kubernetes cluster:

There must be more than one node available at any time.
The control plane must be running on more than one node so that losing a single node would not render the cluster inoperable.
The cluster state must be in a datastore that is itself highly available.

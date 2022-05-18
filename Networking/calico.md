# built in networking with calico

https://thenewstack.io/install-calico-to-enhance-kubernetes-built-in-networking-capability/#:~:text=Calico%2C%20from%20network%20software%20provider,network%20policies%20within%20the%20cluster.


https://www.youtube.com/watch?v=1TsORarOEXc&t=956s

https://projectcalico.docs.tigera.io/getting-started/kubernetes/microk8s

Install calicoctl
 8 MINUTE READ
Big picture
This guide helps you install the calicoctl command line tool to manage Calico resources and perform administrative functions.

Value
The calicoctl command line tool is required in order to use many of Calico’s features. It is used to manage Calico policies and configuration, as well as view detailed cluster status.

Concepts
API groups
All Kubernetes resources belong to an API group. The API group is indicated by the resource’s apiVersion. For example, Calico uses resources in the projectcalico.org/v3 API group for configuration, and the operator uses resources in the operator.tigera.io/v1 API group.

You can read more about API groups in the Kubernetes documentation.

calicoctl and kubectl
In order to manage Calico APIs in the projectcalico.org/v3 API group, you should use calicoctl. This is because calicoctl provides important validation and defaulting for these resources that is not available in kubectl. However, kubectl should still be used to manage other Kubernetes resources.

Note: If you would like to use kubectl to manage projectcalico.org/v3 API resources, you can use the Calico API server.

Warning: Never modify resources in the crd.projectcalico.org API group directly. These are internal data representations and modifying them directly may result in unexpected behavior.

In addition to resource management, calicoctl also enables other Calico administrative tasks such as viewing IP pool utilization and BGP status.

Datastore
Calico objects are stored in one of two datastores, either etcd or Kubernetes. The choice of datastore is determined at the time Calico is installed. Typically for Kubernetes installations the Kubernetes datastore is the default.

You can run calicoctl on any host with network access to the Calico datastore as either a binary or a container. For step-by-step instructions, refer to the section that corresponds to your desired deployment.


Install calicoctl as a binary on a single host
Log into the host, open a terminal prompt, and navigate to the location where you want to install the binary.

Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/.

Use the following command to download the calicoctl binary.

curl -L https://github.com/projectcalico/calico/releases/download/v3.23.1/calicoctl-linux-amd64 -o calicoctl
Set the file to be executable.

chmod +x ./calicoctl

https://ahmedjama.com/blog/2020/09/getting-started-with-calico-for-k8s/
A managed service provider (MSP) is a third-party company that remotely manages a customer's information technology (IT) infrastructure and end-user systems ...

One of the requirements of the k8s networking model is that all pods running in a host are able to communicate with each other. This flat networking model posses some security challenges in clusters that are multi-tenanted. So for an MSP running multiple customer applications on a single cluster; this can lead to customers accessing each other’s service. Even in a setup where multiple business units share a common cluster having a network security policy is needed from a goverance and isolation perspective. Project calico aims to achieve two main objectives.

It can provide highly scalable CNI based networking plugin for k8s
It can provide network policy solution that can enforce network policies at a pod level
In this post we will stand up a mircok8s cluster on Ubuntu 20.04 and install calico on top of that. Once we have calico installed we will implement network policies to provide pod to pod security policies.

Calico objects are stored in one of two datastores, either etcd or Kubernetes. The choice of datastore is determined at the time Calico is installed. Typically for Kubernetes installations the Kubernetes datastore is the default.

kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calicoctl-etcd.yaml
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calicoctl.yaml

kubectl apply -f https://docs.projectcalico.org/manifests/calicoctl.yaml
pod/calicoctl created

Run the kubectl command below output of the calico profiles
$ kubectl exec -ti -n kube-system calicoctl -- /calicoctl get profiles -o wide


IMPLEMENTING NETWORK POLICY WITH CALICO
Now that we have calico up and running we will implement networking policy using the demo on the calico website (https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/kubernetes-demo)

Create the frontend, backend, client, and management-ui apps.
$ kubectl create -f https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/manifests/00-namespace.yaml
$ kubectl create -f https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/manifests/01-management-ui.yaml
$ kubectl create -f https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/manifests/02-backend.yaml
$ kubectl create -f https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/manifests/03-frontend.yaml
$ kubectl create -f https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/manifests/04-client.yaml

https://docs.projectcalico.org/security/tutorials/kubernetes-policy-demo/kubernetes-demo


Optional command line interface (CLI) to manage Calico resources.

The calicoctl CLI tool allows management of Calico API resources, and can be used to perform other administrative tasks for managing a Calico installation.

You can use kubectl to manage Calico resources instead by installing the Calico API server.

https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/


Debugging DNS Resolution
This page provides hints on diagnosing DNS problems.

Before you begin
You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using minikube or you can use one of these Kubernetes playgrounds:

Katacoda
Play with Kubernetes

Your cluster must be configured to use the CoreDNS addon or its precursor, kube-dns.
Your Kubernetes server must be at or later than version v1.6. To check the version, enter kubectl version.

Create a simple Pod to use as a test environment
admin/dns/dnsutils.yaml Copy admin/dns/dnsutils.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: k8s.gcr.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always








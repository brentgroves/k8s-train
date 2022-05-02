Kubernetes on a Bare Metal Host with MicroK8s
https://zero-to-jupyterhub.readthedocs.io/en/latest/kubernetes/other-infrastructure/step-zero-microk8s.html
If you have server hardware available and a small enough user base it’s possible to use Canonical’s MicroK8s in place of a cloud vendor.
With no ability to scale, users will not be able to access their notebooks when memory and CPU resources are exhausted. Read the section on resource planning and set resource limits accordingly.

A cloud provider such as Google Cloud, Microsoft Azure, Amazon EC2, IBM Cloud…
Kubernetes to manage resources on the cloud
Helm v3 to configure and control the packaged JupyterHub installation
JupyterHub to give users access to a Jupyter computing environment
A terminal interface on some operating system

helm, the package manager for Kubernetes, is a useful command line tool for: installing, upgrading and managing applications on a Kubernetes cluster. Helm packages are called charts. We will be installing and managing JupyterHub on our Kubernetes cluster using a Helm chart.

Charts are abstractions describing how to install packages onto a Kubernetes cluster. When a chart is deployed, it works as a templating engine to populate multiple yaml files for package dependencies with the required variables, and then runs kubectl apply to apply the configuration to the resource and install the package.

Any questions:
If you have questions, please:

  1. Read the guide at https://z2jh.jupyter.org
  2. Ask for help or chat to us on https://discourse.jupyter.org/
  3. If you find a bug please report it at https://github.com/jupyterhub/zero-to-jupyterhub-k8s/issues

How do find releases.
Your release is named "helm-release-frt" and installed into the namespace "k8s-namespace-frt".
helm list --namespace=k8s-namespace-frt
helm list --all-namespaces

How do you find all the resources managed by a release?
 kubectl --namespace=k8s-namespace-frt get all // this is the best way unless your chart has recommended labels

helm get all helm-release-frt --namespace=k8s-namespace-frt // This return too much info
Answer is to use K8s label.
What is a label? It is used by Kubernetes to identify this resource
Helm itself never requires that a particular label be present. Labels that are marked REC are recommended, and should be placed onto a chart for global

To list all resources managed by the helm and part of a specific release: (edit release-name)
kubectl get all --all-namespaces -l='app.kubernetes.io/managed-by=Helm,app.kubernetes.io/instance=release-name'
Update: Labels key may vary over time, follow the official documentation for the latest labels.

kubectl get all --all-namespaces -l='app.kubernetes.io/instance=helm-release-frt'
kubectl get all --all-namespaces -l='helm.sh/chart=jupyterhub-1.2.0'

You can check whether the hub and proxy are ready by running:
 kubectl --namespace=k8s-namespace-frt get pod

To get full information about the JupyterHub proxy service run:
  kubectl --namespace=k8s-namespace-frt get svc proxy-public

You can find the public (load-balancekubectl --namespace=k8s-namespace-frt get allr) IP of JupyterHub by running:
  kubectl -n k8s-namespace-frt get svc proxy-public -o jsonpath='{.status.loadBalancer.ingress[].ip}'

how do tell helm what repo to get chart from? jupyterhub/jupyterhub

How to install a release:
helm upgrade --cleanup-on-fail \
  --install helm-release-frt jupyterhub/jupyterhub \
  --namespace k8s-namespace-frt \
  --create-namespace \
  --version=1.2.0 \
  --values config.yaml
helm upgrade --cleanup-on-fail   --install helm-release-frt jupyterhub/jupyterhub   --namespace k8s-namespace-frt   --create-namespace   --values config.yaml

How to list releases? helm list --namespace=k8s-namespace-frt
helm list --all-namespaces

How to update a deployment:
Make changes to config.yml file.
Then run:
helm upgrade --cleanup-on-fail \
>     helm-release-frt jupyterhub/jupyterhub \
>   --namespace k8s-namespace-frt \
>   --version=1.2.0 \
>   --values config.yaml
How to update a release? helm upgrade --cleanup-on-fail   helm-release-frt jupyterhub/jupyterhub   --namespace k8s-namespace-frt   --version=1.2.0
helm upgrade --cleanup-on-fail \
    helm-release-frt jupyterhub/jupyterhub \
  --namespace k8s-namespace-frt \
  --version=1.2.0 \
  --values config.yaml


How do you uninstall a release?
kubectl --namespace=k8s-namespace-frt get all // check the resources before the uninstall
It removes all of the resources associated with the last release of the chart as well as the release history, freeing it up for future use.
helm uninstall helm-release-frt --namespace k8s-namespace-frt
kubectl --namespace=k8s-namespace-frt get all  // make sure all the resources were removed.
I noticed everything was deleted except the running pod.
so delete the namespace to delete the pod.
kubectl delete namespace k8s-namespace-frt


How do you delete a namespace?
kubectl delete namespace k8s-namespace-frt


What is the difference between uninstalling a release and deleting a namespace?
Doing helm uninstall ...  won't just remove the pod, but it will remove all the resources created by helm when it installed the chart. For a single pod, this might not be any different to using kubectl delete... but when you have tens or hundreds of different resources and dependent charts, doing all this manually by doing kubectl delete... becomes cumbersome, time-consuming and error-prone.

Generally if you're deleting something off the cluster, use the same method you used to install it in in the first place. If you used helm to install it into the cluster, use helm to remove it. If you used kubectl create or kubectl apply, use kubectl delete to remove it.
I will add a point that we use, quite a lot. helm uninstall/install/upgrade has hooks attached to its lifecycle. This matters a lot, here is a small example.

We have database scripts that are run as part of a job. Say you prepare a release with version 1.2.3 and as part of that release you add a column in a table - you have a script for that (liquibase/flyway whatever) that will run automatically when the chart is installed. In plain english helm install allows you to say in this case : "before installing the code, upgrade the DB schema". This is awesome and allows you to tie the lifecycle of such scripts, to the lifecycle of the chart.

The same works for downgrade, you could say that when you downgrade, revert the schema, or take any needed action. kubectl delete simply does not have such functionality.

docker run -it --rm \
    -p 8888:8888 \
    --user root \
    -e NB_USER="my-username" \
    -e CHOWN_HOME=yes \
    -w "/home/${NB_USER}" \
    jupyter/base-notebook

How do you run jupyterhub as root?
docker run -it --rm \
-p 8888:8888 \
--user root \
-e GRANT_SUDO=yes
jupyter/all-spark-notebook:2022-04-04

    Permission-specific configurations
-e NB_UMASK=<umask> - Configures Jupyter to use a different umask value from default, i.e. 022. For example, if setting umask to 002, new files will be readable and writable by group members instead of the owner only. Check this Wikipedia article for an in-depth description of umask and suitable values for multiple needs. While the default umask value should be sufficient for most use cases, you can set the NB_UMASK value to fit your requirements.

# Driver=/opt/microsoft/msodbcsql17/lib64/libmsodbcsql-17.9.so.1.1

-e GRANT_SUDO=yes - Instructs the startup script to grant the NB_USER user passwordless sudo capability. 
You do not need this option to allow the user to conda or pip install additional packages. 
This option is helpful for cases when you wish to give ${NB_USER} the ability to install OS packages with apt 
or modify other root-owned files in the container. You must run the container with --user root for this option to take effect. 
(The start-notebook.sh script will su ${NB_USER} after adding ${NB_USER} to sudoers.) 
You should only enable sudo if you trust the user or if the container is running on an isolated host.
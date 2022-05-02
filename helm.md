#Kustomize and #Helm serve the same primary function. Both allow us to define applications in a more flexible way than using only #Kubernetes manifests. However, the way Helm solves the problem is quite different from the approach adopted with Kustomize. Which one is better? Which one should you choose?
https://www.youtube.com/watch?v=Q19x62ZrR-U

https://helm.sh/docs/chart_template_guide/getting_started/

Render chart templates locally and display the output.

Any values that would normally be looked up or retrieved in-cluster will be faked locally. Additionally, none of the server-side testing of chart validity (e.g. whether an API is supported) is done.




1. if you are using pre-built software or you are deploying to multiple environments there is going to be some varialbe
you need to parametize. So it is inadvisable to rely on a static
helm has more features than Kustomize
Learn about helm:
https://helm.sh/docs/

A Chart is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

A Repository is the place where charts can be collected and shared. It's like Perl's CPAN archive or the Fedora Package Database, but for Kubernetes packages.

A Release is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.
interacts directly with the Kubernetes API server to install, upgrade, query, and remove Kubernetes resources.

How to add a repo:
helm repo add bitnami https://charts.bitnami.com/bitnami
https://artifacthub.io/packages/helm/artifact-hub/artifact-hub
helm repo add artifact-hub https://artifacthub.github.io/helm-charts
helm search repo nginx


How to create a helm chart:
helm create demo
create a new chart with the given name

How to deploy a helm chart:
helm install demo1 ./demo
demo1 is the release name and ./demo is the helm chart created.
How to install jupyternode
helm upgrade --cleanup-on-fail \
  --install helm-release-frt jupyterhub/jupyterhub \
  --namespace k8s-namespace-frt \
  --create-namespace \
  --version=1.2.0 \
  --values config.yaml

how to change
modify config.yaml
then run 
helm upgrade --cleanup-on-fail \
    helm-release-frt jupyterhub/jupyterhub \
  --namespace k8s-namespace-frt \
  --version=1.2.0 \
  --values config.yaml


How to create a gzip file from a helm chart directory
helm package demo/

How to run the templating engine to create a complete manifest file with release name of demo1.
helm template demo1 demo > demo1.yaml

START HERE
We could also retrieve the manifest from a k8s deployment:
helm get manifest demo1


https://prefetch.net/blog/2019/10/16/the-beginners-guide-to-creating-kubernetes-manifests/

So let’s begin with the basics. A Kubernetes manifest describes the resources (e.g., Deployments, Services, Pods, etc.) you want to create, and how you want those resources to run inside a cluster. I will describe how to learn more about each resource type later in this post. When you define a resource in a manifest it will contain the following four fields:

apiVersion: apps/v1
kind: Deployment
metadata:
  ...
spec:
  ...
The apiVersion: field specifies the API group you want to use to create the resource and the version of the API to use. Kubernetes APIs are aggregated into API groups which allows the API server to group APIs by purpose. If we dissect the apiVersion line “apps” would be the API group and v1 would be the version of the apps API to use. To list the available API groups and their versions you can run kubectl with the “api-versions” option:

$ kubectl api-versions |more

The second line, “kind:", lists the type of resource you want to create. Deployments, ReplicaSets, CronJobs, StatefulSet, etc. are examples of resources you can create. You can use the kubectl “api-resources” command to view the available resource types as well as the API group they are associated with:

$ kubectl api-resources |more

With the “api-versions” and “api-resources” commands we can find out the available resources (KIND column), the API group (APIGROUP column) the resource type is associated with, and the API group versions (output from api-versions). This information can be used to fill in the apiVersion: and kind: fields. To understand the purpose of each resource type you can use the kubectl “explain” command:

$ kubectl explain --api-version=apps/v1 replicaset

This will give you a detailed explanation of the resource passed as an argument as well as the fields you can populate. Nifty! Now that we’ve covered the first two fields we can move on to metadata: and spec:. The metadata: section is used to uniquely identify the resource inside a Kubernetes cluster. This is were you name the resource, assign tags, annotations, specify a namespace, etc. To view the fields you can add to the metadata: section you can append the “.metadata” string to the resource type passed to “explain”:

$ kubectl explain deployment.metadata | more

Now that we’ve covered the first 3 fields let’s dig into what makes a manifest ticket. The spec: section! This section describes how to create and manage a resource. You will define the container image to use, the number of replicas in a ReplicaSet, the selector criteria, liveness and readiness probe definitions, etc. here To view the fields you can add to the spec: section you can append the “.spec” string to the resource type passed to explain:

$ kubectl explain deployment.spec | more

Kubectl explain does a really nice job of showing the values under each section, but stitching these together by hand takes time and a lot of patience. To make this process easier the kubectl developers provided the “-o yaml” and “–dry-run” options. These options can be combined with the run and create commands to generate a basic manifest for the resource passed as an argument:

$ kubectl create deployment nginx --image=nginx -o yaml --dry-run

Once you have a basic manifest to work with you can start extending it by adding additional fields to the spec: and metadata: sections. You can also add the “–recursive” option to kubectl explain to get a hierarchical view of the various fields. The following example shows how to recursively show every option you can use to customize the containers field:

$ kubectl explain deployment.spec.template.spec.containers --recursive | more

If you want to learn more about a specific field you can pass it to explain to get more information:

$ kubectl explain deployment.spec.selector.matchExpressions.operator







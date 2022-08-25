https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
The Kubernetes scheduler’s default behavior works well for most cases -- for example, it ensures that pods are only placed on nodes that have sufficient free resources, it ties to spread pods from the same set (ReplicaSet, StatefulSet, etc.) across nodes, it tries to balance out the resource utilization of nodes, etc.

But sometimes you want to control how your pods are scheduled. For example, perhaps you want to ensure that certain pods only schedule on nodes with specialized hardware, or you want to co-locate services that communicate frequently, or you want to dedicate a set of nodes to a particular set of users. Ultimately, you know much more about how your applications should be scheduled and deployed than Kubernetes ever will. So Kubernetes 1.6 offers four advanced scheduling features: node affinity/anti-affinity, taints and tolerations, pod affinity/anti-affinity, and custom schedulers. Each of these features are now in beta in Kubernetes 1.6.

Node Affinity/Anti-Affinity

Node Affinity/Anti-Affinity is one way to set rules on which nodes are selected by the scheduler. This feature is a generalization of the nodeSelector feature which has been in Kubernetes since version 1.0. The rules are defined using the familiar concepts of custom labels on nodes and selectors specified in pods, and they can be either required or preferred, depending on how strictly you want the scheduler to enforce them.

Required rules must be met for a pod to schedule on a particular node. If no node matches the criteria (plus all of the other normal criteria, such as having enough free resources for the pod’s resource request), then the pod won’t be scheduled. Required rules are specified in the requiredDuringSchedulingIgnoredDuringExecution field of nodeAffinity.

For example, if we want to require scheduling on a node that is in the us-central1-a GCE zone of a multi-zone Kubernetes cluster, we can specify the following affinity rule as part of the Pod spec:

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: "failure-domain.beta.kubernetes.io/zone"
              operator: In
              values: ["us-central1-a"]
“IgnoredDuringExecution” means that the pod will still run if labels on a node change and affinity rules are no longer met. There are future plans to offer requiredDuringSchedulingRequiredDuringExecution which will evict pods from nodes as soon as they don’t satisfy the node affinity rule(s).

Preferred rules mean that if nodes match the rules, they will be chosen first, and only if no preferred nodes are available will non-preferred nodes be chosen. You can prefer instead of require that pods are deployed to us-central1-a by slightly changing the pod spec to use preferredDuringSchedulingIgnoredDuringExecution:

  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: "failure-domain.beta.kubernetes.io/zone"
              operator: In
              values: ["us-central1-a"]
Node anti-affinity can be achieved by using negative operators. So for instance if we want our pods to avoid us-central1-a we can do this:

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: "failure-domain.beta.kubernetes.io/zone"
              operator: NotIn
              values: ["us-central1-a"]
Valid operators you can use are In, NotIn, Exists, DoesNotExist. Gt, and Lt.

Additional use cases for this feature are to restrict scheduling based on nodes’ hardware architecture, operating system version, or specialized hardware. Node affinity/anti-affinity is beta in Kubernetes 1.6.
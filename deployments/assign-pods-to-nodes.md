https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
More practical use-cases
Inter-pod affinity and anti-affinity can be even more useful when they are used with higher level collections such as ReplicaSets, StatefulSets, Deployments, etc. These rules allow you to configure that a set of workloads should be co-located in the same defined topology; for example, preferring to place two related Pods onto the same node.

For example: imagine a three-node cluster. You use the cluster to run a web application and also an in-memory cache (such as Redis). For this example, also assume that latency between the web application and the memory cache should be as low as is practical. You could use inter-pod affinity and anti-affinity to co-locate the web servers with the cache as much as possible.

In the following example Deployment for the Redis cache, the replicas get the label app=store. The podAntiAffinity rule tells the scheduler to avoid placing multiple replicas with the app=store label on a single node. This creates each cache in a separate node.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
The following example Deployment for the web servers creates replicas with the label app=web-store. The Pod affinity rule tells the scheduler to place each replica on a node that has a Pod with the label app=store. The Pod anti-affinity rule tells the scheduler never to place multiple app=web-store servers on a single node.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine
Creating the two preceding Deployments results in the following cluster layout, where each web server is co-located with a cache, on three separate nodes.

node-1	node-2	node-3
webserver-1	webserver-2	webserver-3
cache-1	cache-2	cache-3
The overall effect is that each cache instance is likely to be accessed by a single client, that is running on the same node. This approach aims to minimize both skew (imbalanced load) and latency.

You might have other reasons to use Pod anti-affinity. See the ZooKeeper tutorial for an example of a StatefulSet configured with anti-affinity for high availability, using the same technique as this example.




Assigning Pods to Nodes
You can constrain a Pod so that it is restricted to run on particular node(s), or to prefer to run on particular nodes. There are several ways to do this and the recommended approaches all use label selectors to facilitate the selection. Often, you do not need to set any such constraints; the scheduler will automatically do a reasonable placement (for example, spreading your Pods across nodes so as not place Pods on a node with insufficient free resources). However, there are some circumstances where you may want to control which node the Pod deploys to, for example, to ensure that a Pod ends up on a node with an SSD attached to it, or to co-locate Pods from two different services that communicate a lot into the same availability zone.

You can use any of the following methods to choose where Kubernetes schedules specific Pods:

nodeSelector field matching against node labels
Affinity and anti-affinity
nodeName field
Pod topology spread constraints

Node isolation/restriction
Adding labels to nodes allows you to target Pods for scheduling on specific nodes or groups of nodes. You can use this functionality to ensure that specific Pods only run on nodes with certain isolation, security, or regulatory properties.

If you use labels for node isolation, choose label keys that the kubelet cannot modify. This prevents a compromised node from setting those labels on itself so that the scheduler schedules workloads onto the compromised node.

The NodeRestriction admission plugin prevents the kubelet from setting or modifying labels with a node-restriction.kubernetes.io/ prefix.

To make use of that label prefix for node isolation:

Ensure you are using the Node authorizer and have enabled the NodeRestriction admission plugin.
Add labels with the node-restriction.kubernetes.io/ prefix to your nodes, and use those labels in your node selectors. For example, example.com.node-restriction.kubernetes.io/fips=true or example.com.node-restriction.kubernetes.io/pci-dss=true.

nodeSelector 
nodeSelector is the simplest recommended form of node selection constraint. You can add the nodeSelector field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

See Assign Pods to Nodes for more information.

Affinity and anti-affinity
nodeSelector is the simplest way to constrain Pods to nodes with specific labels. Affinity and anti-affinity expands the types of constraints you can define. Some of the benefits of affinity and anti-affinity include:

The affinity/anti-affinity language is more expressive. nodeSelector only selects nodes with all the specified labels. Affinity/anti-affinity gives you more control over the selection logic.
You can indicate that a rule is soft or preferred, so that the scheduler still schedules the Pod even if it can't find a matching node.
You can constrain a Pod using labels on other Pods running on the node (or other topological domain), instead of just node labels, which allows you to define rules for which Pods can be co-located on a node.
The affinity feature consists of two types of affinity:

Node affinity functions like the nodeSelector field but is more expressive and allows you to specify soft rules.
Inter-pod affinity/anti-affinity allows you to constrain Pods against labels on other Pods.
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
Node affinity
Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

requiredDuringSchedulingIgnoredDuringExecution: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
preferredDuringSchedulingIgnoredDuringExecution: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

Note: In the preceding types, IgnoredDuringExecution means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.
You can specify node affinities using the .spec.affinity.nodeAffinity field in your Pod spec.

For example, consider the following Pod spec:

pods/pod-with-node-affinity.yaml Copy pods/pod-with-node-affinity.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0

    In this example, the following rules apply:

The node must have a label with the key topology.kubernetes.io/zone and the value of that label must be either antarctica-east1 or antarctica-west1.
The node preferably has a label with the key another-node-label-key and the value another-node-label-value.
You can use the operator field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use In, NotIn, Exists, DoesNotExist, Gt and Lt.

NotIn and DoesNotExist allow you to define node anti-affinity behavior. Alternatively, you can use node taints to repel Pods from specific nodes.

Note:
If you specify both nodeSelector and nodeAffinity, both must be satisfied for the Pod to be scheduled onto a node.

If you specify multiple nodeSelectorTerms associated with nodeAffinity types, then the Pod can be scheduled onto a node if one of the specified nodeSelectorTerms can be satisfied.

If you specify multiple matchExpressions associated with a single nodeSelectorTerms, then the Pod can be scheduled onto a node only if all the matchExpressions are satisfied.




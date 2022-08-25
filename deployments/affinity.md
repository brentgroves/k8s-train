https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
Assigning Pods to Nodes
You can constrain a Pod so that it is restricted to run on particular node(s), or to prefer to run on particular nodes. There are several ways to do this and the recommended approaches all use label selectors to facilitate the selection. Often, you do not need to set any such constraints; the scheduler will automatically do a reasonable placement (for example, spreading your Pods across nodes so as not place Pods on a node with insufficient free resources). However, there are some circumstances where you may want to control which node the Pod deploys to, for example, to ensure that a Pod ends up on a node with an SSD attached to it, or to co-locate Pods from two different services that communicate a lot into the same availability zone.

You can use any of the following methods to choose where Kubernetes schedules specific Pods:

nodeSelector field matching against node labels
Affinity and anti-affinity
nodeName field
Pod topology spread constraints


https://stackoverflow.com/questions/62292476/deployment-affinity#:~:text=Inter%2Dpod%20affinity%20and%20anti,based%20on%20labels%20on%20nodes.
Kubernetes scheduler will by default try to schedule deployment replicas on different nodes if possible (as long as a node satisfies momory/cpu requirements).
If don't, 2 (or more) pod replicas can get scheduled on one node and you can use several techniques to prevent this.
One of these techniques is called pod affinity. In k8s documentation you can read:
Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled based on labels on pods that are already running on the node rather than based on labels on nodes. The rules are of the form "this pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more pods that meet rule Y"
With podaffinity you need to be aware that if a pod for some reason cannot be scheduled on a node (lack of resources or tainted node) and it will end up in pending state.
You should also remember that when running 3 node cluster (1 master + 2 workers) it is common to have NoSchedule taint on master node (which is typical for clusters created with e.g. kubeadm) that disallows scheduling pods on master node.
If this applies to you and you still want to schedult pods on mater node, you need to either delete NoSchedule taint:
kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
Or use toleration:
apiVersion: extensions/v1beta1
kind: Deployment
  spec:
    spec:
      tolerations:
        - key: "node-role.kubernetes.io/master"
          effect: "NoSchedule"
          operator: "Exists"
In comments @suren mentioned DaemonSets which can be used in some cases but when scaling your cluster, your application will scale with it, and it may not be desired.

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


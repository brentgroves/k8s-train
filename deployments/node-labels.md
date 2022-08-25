Node labels 
Like many other Kubernetes objects, nodes have labels. You can attach labels manually. Kubernetes also populates a standard set of labels on all nodes in a cluster. See Well-Known Labels, Annotations and Taints for a list of common node labels.

Note: The value of these labels is cloud provider specific and is not guaranteed to be reliable. For example, the value of kubernetes.io/hostname may be the same as the node name in some environments and a different value in other environments.
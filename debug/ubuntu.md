https://downey.io/notes/dev/ubuntu-sleep-pod-yaml/

Sometimes it can be helpful to deploy a simple Ubuntu container to a cluster when debugging. Say you just applied some new NetworkPolicy and want to test network connectivity between namespaces. Or maybe you added a new mutating admission webhook to inject sidecar containers and you need something to test it out with. Or maybe you just want a sandbox container to deploy and play around in.

One thing I like to do is deploy a Pod running Ubuntu that will let me install whatever tools I want. No need to worry about thin, distroless images that are so secure I can’t do anything! With the Ubuntu image everything is just an apt install away. 😌

However, it’s not as simple as running the ubuntu image on its own. You need to make it actually do something or the container will just exit immediately. Fortunately this is easy enough… just make the container sleep for a long time!

I do this fairly often and hate having to write the YAML from scratch everytime. So this post will serve as a breadcrumb for my future self to find and copy and paste from in the future. 🤞

the yaml
The following YAML will deploy a Pod with a container running the ubuntu Docker image that sleeps for a week. Plenty of time to do what you need!

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "604800"
    imagePullPolicy: IfNotPresent
    name: ubuntu
  restartPolicy: Always
applying the yaml
You can apply this via the following by piping stdin to kubectl:

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
  - image: ubuntu
    command:
      - "sleep"
      - "604800"
    imagePullPolicy: IfNotPresent
    name: ubuntu
  restartPolicy: Always
EOF
Or you can apply the raw contents of this Gist directly:
https://computingforgeeks.com/deploy-ubuntu-pod-in-kubernetes-openshift/

kubectl apply -f https://gist.githubusercontent.com/tcdowney/b8a0297241b74f94ef1fc6627f7ea69a/raw/eaae035f5adca37ca00d4a49f1c1958fe3db89e3/ubuntu-sleep.yaml

using the pod
Start up an interactive shell in the container:

$ kubectl exec -it ubuntu -- /bin/bash

root@ubuntu:/#
Now you can install whatever you want! For example, I often install curl via the following:

$ apt update && apt install curl -y
what about ephemeral debug containers?
If you’ve been following along with the latest Kubernetes releases, you may be aware of a new alpha feature in Kubernetes 1.18 known as ephemeral debug containers. This features lets you take a running Pod and attach an arbitrary “debug” container that has all of the tools you might need to debug it. This is really powerful for several reasons:

If a Pod is misbehaving you can attach the container to it and see what’s going on directly.
You can continue to follow best practices and publish small container images. No need to include debug utilities “just in case.”
No need to look up this page to copy paste some YAML for a hacky Ubuntu sleep pod!
I’m really looking forward to them. However, Kubernetes 1.18 is still pretty bleeding age (at least at the time of writing this post) and the feature is still in alpha. There’s also some use cases for the Ubuntu pod that it doesn’t cover so this method still has some life in it yet!
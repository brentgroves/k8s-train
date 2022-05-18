https://kubernetes.io/docs/tasks/inject-data-application/_print/

Define an environment variable for a container
When you create a Pod, you can set environment variables for the containers that run in the Pod. To set environment variables, include the env or envFrom field in the configuration file.

In this exercise, you create a Pod that runs one container. The configuration file for the Pod defines an environment variable with name DEMO_GREETING and value "Hello from the environment". Here is the configuration manifest for the Pod:

pods/inject/envars.yaml Copy pods/inject/envars.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
Create a Pod based on that manifest:

kubectl apply -f https://k8s.io/examples/pods/inject/envars.yaml
List the running Pods:

kubectl get pods -l purpose=demonstrate-envars
The output is similar to:

NAME            READY     STATUS    RESTARTS   AGE
envar-demo      1/1       Running   0          9s
List the Pod's container environment variables:

kubectl exec envar-demo -- printenv
The output is similar to this:

NODE_VERSION=4.4.2
EXAMPLE_SERVICE_PORT_8080_TCP_ADDR=10.3.245.237
HOSTNAME=envar-demo
...
DEMO_GREETING=Hello from the environment
DEMO_FAREWELL=Such a sweet sorrow


Note: The environment variables set using the env or envFrom field override any environment variables specified in the container image.
Note: Environment variables may reference each other, however ordering is important. Variables making use of others defined in the same context must come later in the list. Similarly, avoid circular references.
Using environment variables inside of your config
Environment variables that you define in a Pod's configuration can be used elsewhere in the configuration, for example in commands and arguments that you set for the Pod's containers. In the example configuration below, the GREETING, HONORIFIC, and NAME environment variables are set to Warm greetings to, The Most Honorable, and Kubernetes, respectively. Those environment variables are then used in the CLI arguments passed to the env-print-demo container.

apiVersion: v1
kind: Pod
metadata:
  name: print-greeting
spec:
  containers:
  - name: env-print-demo
    image: bash
    env:
    - name: GREETING
      value: "Warm greetings to"
    - name: HONORIFIC
      value: "The Most Honorable"
    - name: NAME
      value: "Kubernetes"
    command: ["echo"]
    args: ["$(GREETING) $(HONORIFIC) $(NAME)"]
Upon creation, the command echo Warm greetings to The Most Honorable Kubernetes is run on the container.

The Downward API
There are two ways to expose Pod and Container fields to a running Container:

Environment variables
Volume Files
Together, these two ways of exposing Pod and Container fields are called the Downward API.

Use Pod fields as values for environment variables
In this exercise, you create a Pod that has one Container. Here is the configuration file for the Pod:

pods/inject/dapi-envars-pod.yaml Copy pods/inject/dapi-envars-pod.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-fieldref
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
          printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
          sleep 10;
        done;
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never

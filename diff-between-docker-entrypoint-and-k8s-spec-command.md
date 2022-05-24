https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#notes
Basically the COMMAND can override what is mentioned in the docker ENTRYPOINT

Simple example:

To override the dockerfile ENTRYPOINT, just add these fields to your K8s template (Look at the command and args):

apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["/bin/sh"]
    args: ["-c", "printenv; #OR WHATEVER COMMAND YOU WANT"]
  restartPolicy: OnFailure

https://stackoverflow.com/questions/44316361/difference-between-docker-entrypoint-and-kubernetes-container-spec-command

Dockerfile has a parameter for ENTRYPOINT and while writing Kubernetes deployment YAML file, there is a parameter in Container spec for COMMAND.

Kubernetes provides us with multiple options on how to use these commands:

When you override the default Entrypoint and Cmd in Kubernetes .yaml file, these rules apply:

If you do not supply command or args for a Container, the defaults defined in the Docker image are used.
If you supply only args for a Container, the default Entrypoint defined in the Docker image is run with the args that you supplied.
If you supply a command for a Container, only the supplied command is used. The default EntryPoint and the default Cmd defined in the Docker image are ignored. Your command is run with the args supplied (or no args if none supplied).

Here is an example:

Dockerfile:

FROM alpine:latest
COPY "executable_file" /
ENTRYPOINT [ "./executable_file" ]
Kubernetes yaml file:

 spec:
    containers:
      - name: container_name
        image: image_name
        args: ["arg1", "arg2", "arg3"]
https://stackoverflow.com/questions/33979501/kubernetes-passing-multiple-commands-to-the-container

https://yaml-multiline.info/
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:  # specification of the pod’s contents
  restartPolicy: Never
  containers:
  - name: hello
    image: "ubuntu:14.04"
    command: ["/bin/sh"]
    args:
      - -c
      - >-
          command1 arg1 arg2 &&
          command2 arg3 &&
          command3 arg4

83

There can only be a single entrypoint in a container... if you want to run multiple commands like that, make bash be the entry point, and make all the other commands be an argument for bash to run:

command: ["/bin/bash","-c","touch /foo && echo 'here' && ls /"]   


# cron start
    # command: ["/bin/sh"]
    # args: ["-c", "printenv; #OR WHATEVER COMMAND YOU WANT"]    
# command: ["/bin/sh","-c"]
# args: ["command one; command two && command three"]

    # command: ["/bin/sh"]
    # args:
    #   - -c
    #   - >-
    #       command1 arg1 arg2 &&
    #       command2 arg3 &&
    #       command3 arg4    

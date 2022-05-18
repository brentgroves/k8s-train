https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

Configure Liveness, Readiness and Startup Probes
This page shows how to configure liveness, readiness and startup probes for containers.

The kubelet uses liveness probes to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.

The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

The kubelet uses startup probes to know when a container application has started. If such a probe is configured, it disables liveness and readiness checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

Define a liveness command 
Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.

In this exercise, you create a Pod that runs a container based on the k8s.gcr.io/busybox image. Here is the configuration file for the Pod:

pods/probe/exec-liveness.yaml Copy pods/probe/exec-liveness.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

      In the configuration file, you can see that the Pod has a single Container. The periodSeconds field specifies that the kubelet should perform a liveness probe every 5 seconds. The initialDelaySeconds field tells the kubelet that it should wait 5 seconds before performing the first probe. To perform a probe, the kubelet executes the command cat /tmp/healthy in the target container. If the command succeeds, it returns 0, and the kubelet considers the container to be alive and healthy. If the command returns a non-zero value, the kubelet kills the container and restarts it.

When the container starts, it executes this command:

/bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"
For the first 30 seconds of the container's life, there is a /tmp/healthy file. So during the first 30 seconds, the command cat /tmp/healthy returns a success code. After 30 seconds, cat /tmp/healthy returns a failure code.

Create the Pod:

kubectl apply -f https://k8s.io/examples/pods/probe/exec-liveness.yaml
Within 30 seconds, view the Pod events:

kubectl describe pod liveness-exec
The output indicates that no liveness probes have failed yet:

Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11s   default-scheduler  Successfully assigned default/liveness-exec to node01
  Normal  Pulling    9s    kubelet, node01    Pulling image "k8s.gcr.io/busybox"
  Normal  Pulled     7s    kubelet, node01    Successfully pulled image "k8s.gcr.io/busybox"
  Normal  Created    7s    kubelet, node01    Created container liveness
  Normal  Started    7s    kubelet, node01    Started container liveness
After 35 seconds, view the Pod events again:

kubectl describe pod liveness-exec
At the bottom of the output, there are messages indicating that the liveness probes have failed, and the containers have been killed and recreated.

  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  57s                default-scheduler  Successfully assigned default/liveness-exec to node01
  Normal   Pulling    55s                kubelet, node01    Pulling image "k8s.gcr.io/busybox"
  Normal   Pulled     53s                kubelet, node01    Successfully pulled image "k8s.gcr.io/busybox"
  Normal   Created    53s                kubelet, node01    Created container liveness
  Normal   Started    53s                kubelet, node01    Started container liveness
  Warning  Unhealthy  10s (x3 over 20s)  kubelet, node01    Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    10s                kubelet, node01    Container liveness failed liveness probe, will be restarted
Wait another 30 seconds, and verify that the container has been restarted:

kubectl get pod liveness-exec
The output shows that RESTARTS has been incremented:

NAME            READY     STATUS    RESTARTS   AGE
liveness-exec   1/1       Running   1          1m

Define a liveness HTTP request
Another kind of liveness probe uses an HTTP GET request. Here is the configuration file for a Pod that runs a container based on the k8s.gcr.io/liveness image.

pods/probe/http-liveness.yaml Copy pods/probe/http-liveness.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3

In the configuration file, you can see that the Pod has a single container. The periodSeconds field specifies that the kubelet should perform a liveness probe every 3 seconds. The initialDelaySeconds field tells the kubelet that it should wait 3 seconds before performing the first probe. To perform a probe, the kubelet sends an HTTP GET request to the server that is running in the container and listening on port 8080. If the handler for the server's /healthz path returns a success code, the kubelet considers the container to be alive and healthy. If the handler returns a failure code, the kubelet kills the container and restarts it.

Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.

You can see the source code for the server in server.go.

For the first 10 seconds that the container is alive, the /healthz handler returns a status of 200. After that, the handler returns a status of 500.

http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})

The kubelet starts performing health checks 3 seconds after the container starts. So the first couple of health checks will succeed. But after 10 seconds, the health checks will fail, and the kubelet will kill and restart the container.

To try the HTTP liveness check, create a Pod:

kubectl apply -f https://k8s.io/examples/pods/probe/http-liveness.yaml
After 10 seconds, view Pod events to verify that liveness probes have failed and the container has been restarted:

kubectl describe pod liveness-http
In releases prior to v1.13 (including v1.13), if the environment variable http_proxy (or HTTP_PROXY) is set on the node where a Pod is running, the HTTP liveness probe uses that proxy. In releases after v1.13, local HTTP proxy environment variable settings do not affect the HTTP liveness probe.

Define a TCP liveness probe
A third type of liveness probe uses a TCP socket. With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, the container is considered healthy, if it can't it is considered a failure.

pods/probe/tcp-liveness-readiness.yaml Copy pods/probe/tcp-liveness-readiness.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
As you can see, configuration for a TCP check is quite similar to an HTTP check. This example uses both readiness and liveness probes. The kubelet will send the first readiness probe 5 seconds after the container starts. This will attempt to connect to the goproxy container on port 8080. If the probe succeeds, the Pod will be marked as ready. The kubelet will continue to run this check every 10 seconds.

In addition to the readiness probe, this configuration includes a liveness probe. The kubelet will run the first liveness probe 15 seconds after the container starts. Similar to the readiness probe, this will attempt to connect to the goproxy container on port 8080. If the liveness probe fails, the container will be restarted.

To try the TCP liveness check, create a Pod:

kubectl apply -f https://k8s.io/examples/pods/probe/tcp-liveness-readiness.yaml
After 15 seconds, view Pod events to verify that liveness probes:

kubectl describe pod goproxy




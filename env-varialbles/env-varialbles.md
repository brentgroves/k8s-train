

apiVersion: apps/v1
kind: Deployment
metadata:
  name: golang-hello-world-web
  # allow for override: kubectl -n <ns> apply ...
  #namespace: default
  labels:
    app: golang-hello-world-web
spec:
  selector:
    matchLabels:
      app: golang-hello-world-web
  # kubectl scale --replicas=x deployment/golang-hello-world-web
  replicas: 1
  template:
    metadata:
      labels:
        app: golang-hello-world-web
      # skip any istio auto-injection
      annotations:
        sidecar.istio.io/inject: "false"
    spec:

      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - golang-hello-world-web
            topologyKey: "kubernetes.io/hostname"

      containers:
      - name: golang-hello-world-web
        image: fabianlee/docker-golang-hello-world-web:1.0.0
        env:

        # default variables available to be overridden
        - name: APP_CONTEXT
          # if set to /test/' would only deliver from that path
          value: "/myhello/"
        - name: PORT
          value: "8080"

        # Downward API support - inserts these into container as env variables
        # https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#use-pod-fields-as-values-for-environment-variables
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

        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3

        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 3

      restartPolicy: Always

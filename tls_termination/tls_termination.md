https://kubernetes.github.io/ingress-nginx/examples/tls-termination/

https://kubernetes.github.io/ingress-nginx/examples/tls-termination/

https://kubernetes.github.io/ingress-nginx/user-guide/tls/

TLS termination¶
This example demonstrates how to terminate TLS through the nginx Ingress controller.

Prerequisites¶
You need a TLS cert and a test HTTP service for this example.

Deployment¶
Create a ingress.yaml file.


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-test
spec:
  tls:
    - hosts:
      - foo.bar.com
      # This assumes tls-secret exists and the SSL
      # certificate contains a CN for foo.bar.com
      secretName: tls-secret
  ingressClassName: nginx
  rules:
    - host: foo.bar.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            # This assumes http-svc exists and routes to healthy endpoints
            service:
              name: http-svc
              port:
                number: 80
The following command instructs the controller to terminate traffic using the provided TLS cert, and forward un-encrypted HTTP traffic to the test HTTP service.


kubectl apply -f ingress.yaml
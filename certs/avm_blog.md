https://medium.com/avmconsulting-blog/how-to-secure-applications-on-kubernetes-ssl-tls-certificates-8f7f5751d788

Self Signed Certificate
A self-signed SSL certificate is an SSL Certificate that is issued by the person creating it rather than a trusted certificate authority. This can be good for testing environments.

Step 1: Generate a CA private key

$ OpenSSL genrsa -out ca.key 2048

Step 2: Create a self-signed certificate, valid for 365 days.

$ openssl req -x509 \
  -new -nodes  \
  -days 365 \
  -key ca.key \
  -out ca.crt \
  -subj "/CN=yourdomain.com"

Create TLS Secret:
Using kubectl:
$ kubectl create secret tls my-tls-secret \
--key < private key filename> \
--cert < certificate filename>

Using YAML file:

--- 
apiVersion: v1
data: 
  tls.crt: "base64 encoded cert"
  tls.key: "base64 encoded key"
kind: Secret
metadata: 
  name: my-tls-secret
  namespace: default
type: kubernetes.io/tls

Check the secret.

$ kubectl get secrets/my-tls-secret

Describe the secret.

$ kubectl describe secrets/my-tls-secret

Drawbacks of Self-signed Certificate:

Browser Warnings:

Self-signed shows untrusted connection warning while connecting to HTTPS website. Thus, the Self-Signed certificate is not the right choice for online businesses

Authentication Issue:

A self-signed certificate does lack authenticity hence, attackers can replace the self-signed certificate with the attacker’s certificate. However, browsers will have no idea whether it is communicating with the right SSL certificate or a replaced certificate.

How to do TLS between microservices in Kubernetes?
Kubernetes Services and DNS Discovery
In general, it is recommended to put a Service in front of a Deployment that manages pods in Kubernetes. The Service creates a stable DNS and IP endpoint for pods that may be deleted and be assigned a different IP address when recreated. DNS service discovery is automatically enabled with a ClusterIP type service and is in the format: <service name>.<kubernetes namespace>.svc.<cluster domain> where cluster domain is usually cluster.local. This means that we can use the auto-created DNS and assigned ClusterIP in our alt names for our certificate.

Kubernetes Internal CA
Kubernetes does have an internal CA along with API methods to post CSRs and have those CSRs signed by the CA however I would not use the internal CA for securing microservices. The internal CA is primarily used by the kubelet and other internal cluster processes to authenticate to the Kubernetes API server. There is no functionality for autorenewal and I think the cert will always be signed for 30 days.

What are alt names?
https://www.digicert.com/faq/subject-alternative-name.htm#:~:text=The%20Subject%20Alternative%20Name%20Field,Extend%20Validation%20Multi%2DDomain%20Certificate.

Manually Create and Deploy Certificates
You should be able to achieve the same result using your “without Kubernetes” approach using cfssl:
generate CA using cfssl
add CA as trusted in the image (using your Dockerfile approach)
create Kubernetes Service (for example purposes I will use kubectl create)

$ kubectl create service clusterip grpcserver — tcp=8000
describe the created Kubernetes Service, note IP will most likely be different in your case

$ kubectl describe service/grpcserver
Name: grpcserver
Namespace: default
Labels: app=grpcserver
Annotations: <none>
Selector: app=grpcserver
Type: ClusterIP
IP: 10.108.125.158
Port: 8000 8000/TCP
TargetPort: 8000/TCP
Endpoints: <none>
Session Affinity: None
Events: <none>

generate a certificate for gRPCServer with a CN of grpcserver.default.svc.cluster.local the following alt names:

grpcserver
grpcserver.default.svc
grpcserver.default.svc.cluster.local
10.108.125.158
generate the client certificate with cfssl

https://blog.cloudflare.com/introducing-cfssl/


put both certificates into Secret objects

kubectl create secret tls server — cert=server.pem — key=server.key
kubectl create secret tls client — cert=client.pem — key=client.key

Ingress with TLS:
--- 
apiVersion: extensions/v1beta1
kind: Ingress
metadata: 
  name: tls-example-ingress
spec: 
  rules: 
    - 
      host: mydomain.com
      http: 
        paths: 
          - 
            backend: 
              serviceName: my-service
              servicePort: 80
            path: /
  tls: 
    - 
      hosts: 
        - mydomain.com
      secretName: my-tls-secret


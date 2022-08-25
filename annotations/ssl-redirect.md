
https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md#server-side-https-enforcement-through-redirect

Server-side HTTPS enforcement through redirect
By default the controller redirects (308) to HTTPS if TLS is enabled for that ingress. If you want to disable this behavior globally, you can use ssl-redirect: "false" in the NGINX ConfigMap.

To configure this feature for specific ingress resources, you can use the nginx.ingress.kubernetes.io/ssl-redirect: "false" annotation in the particular resource.

When using SSL offloading outside of cluster (e.g. AWS ELB) it may be useful to enforce a redirect to HTTPS even when there is no TLS certificate available. This can be achieved by using the nginx.ingress.kubernetes.io/force-ssl-redirect: "true" annotation in the particular resource.

To preserve the trailing slash in the URI with ssl-redirect, set nginx.ingress.kubernetes.io/preserve-trailing-slash: "true" annotation for that particular resource.

Redirect from/to www
In some scenarios is required to redirect from www.domain.com to domain.com or vice versa. To enable this feature use the annotation nginx.ingress.kubernetes.io/from-to-www-redirect: "true"

!!! attention If at some point a new Ingress is created with a host equal to one of the options (like domain.com) the annotation will be omitted.

!!! attention For HTTPS to HTTPS redirects is mandatory the SSL Certificate defined in the Secret, located in the TLS section of Ingress, contains both FQDN in the common name of the certificate.


https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

Which service should we use for Moxa traffic?
NodePort
A NodePort service is the most primitive way to get external traffic directly to your service. NodePort, as the name implies, opens a specific port on all the Nodes (the VMs), and any traffic that is sent to this port is forwarded to the service.
Why?  Each plant could be assigned there own VM to point to.
Downside: If one of those VM goes down traffic will be lost.

LoadBalancer
When would you use this?
If you want to directly expose a service, this is the default method. All traffic on the port you specify will be forwarded to the service. There is no filtering, no routing, etc. This means you can send almost any kind of traffic to it, like HTTP, TCP, UDP, Websockets, gRPC, or whatever.
The big downside is that each service you expose with a LoadBalancer will get its own IP address, and you have to pay for a LoadBalancer per exposed service, which can get expensive!

Why should we use this?
High Availability.  If one node goes down this will still work. The bottle neck is the 1 IP address, but the service can distribute the traffic to pods on many nodes.
# internal ip of primary pod
primaryPodIP=$(microk8s kubectl get pods -l app=golang-hello-world-web -o=jsonpath="{.items[0].status.podIPs[0].ip}")
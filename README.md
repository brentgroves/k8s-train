# k8s-train
https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/
kubectl describe deployment rss-site
kubectl delete deploy rss-site



https://www.freecodecamp.org/news/what-is-a-helm-chart-tutorial-for-kubernetes-beginners/
helm upgrade --install --create-namespace rss-site ./rss-chart/chart \
  --set image.tag=v1.0.0 \
  --set env=production \
  --set environment.SENDGRID_APIKEY=myKey \
  --set environment.DEFAULT_FROM_ADDRESS="my@email.com" \
  --set environment.DEFAULT_FROM_NAME="Lucas Santos"
docker build --tag brentgroves/etl-pod:15 --build-arg CACHEBUST=$(date +%s) .

docker login

docker push brentgroves/etl-pod:15

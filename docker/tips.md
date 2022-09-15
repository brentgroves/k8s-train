<!-- Stop containers then remove images -->
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q) 

docker rmi $(docker images -qa -f 'dangling=true')

Remove all dangling images 
docker image prune

Remove all dangling images and regular images 
docker image prune -a

docker rmi $(docker images -qa -f 'dangling=true')

docker build --tag brentgroves/etl-pod:15 --build-arg CACHEBUST=$(date +%s) .

docker push brentgroves/etl-pod:15

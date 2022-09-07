docker build --tag brentgroves/etl-pod:3 --build-arg CACHEBUST=$(date +%s) .
# start a container in the background
docker run --name etl-pod -d brentgroves/etl-pod:3
docker container ls -a

# Next, execute a command on the container.
docker exec -it etl-pod pwd
docker exec -it etl-pod pgrep cron

# Next, execute an interactive bash shell on the container.
docker exec -it etl-pod bash


WARNING! This will remove all images without at least one container associated to them.
WARNING! This will remove all images without at least one container associated to them.
docker image prune -a



docker rmi $(docker images -qa -f 'dangling=true')


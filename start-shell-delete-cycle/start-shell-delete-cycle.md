# start a container and watch tail /var/log/cron.log
# as it populates.
docker run --name cron-test -d brentgroves/cron-test:7


# start a container in the background
docker run --name cron-test -d brentgroves/cron-test:7

# Next, execute a command on the container.
docker exec -d cron-test chmod 666 /var/log/cron.log && cat /dev/null > /var/log/cron.log

# Next, execute an interactive bash shell on the container.
docker exec -it cron-test bash

# delete container
docker container rm --force 3fc31553840d

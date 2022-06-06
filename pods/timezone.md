https://dev.to/0xbf/set-timezone-in-your-docker-image-d22

Ubuntu
When your image/root-image is based on Ubuntu, use:
RUN apt-get update && \
    apt-get install -yq tzdata && \
    ln -fs /usr/share/zoneinfo/America/Indiana/Indianapolis /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata

The article states you may have to set TZ also
<!-- https://dev.to/0xbf/set-timezone-in-your-docker-image-d22 # ENV TZ="America/Indiana/Indianapolis"     -->
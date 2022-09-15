# docker-timezones
git clone git@github.com:brentgroves/docker-timezones.git
https://blog.game-changing.de/how-to-set-timezone-and-locale-in-an-ubuntu-image-properly/
The image can be build through docker build -t ubuntu-locale-and-timezone:latest .

There’s a lot of confusion when it comes to setting timezone and locales in a Ubuntu docker container. Here’s my take:

FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

ENV TZ Europe/Berlin

RUN apt-get update \ 
    && apt-get install -yq tzdata locales \
    && rm -rf /var/lib/apt/lists/* \    
    && sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen \
    && locale-gen \
    && ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
    && dpkg-reconfigure tzdata 

ENV LANG en_US.UTF-8  
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

CMD ["/bin/bash"]
The timezone is set by environment variable TZ to Europe/Berlin. Afterwards we install package tzdata, set /etc/localtime, and trigger the system configuration through dpkg-reconfigure tzdata.

Environment variables for locales and language need to be set after the installation of package locales. Within the package installation block we add the desired locales to /etc/locale.gen, and trigger the generation of the system files through locale-gen.

The image can be build through docker build -t ubuntu-locale-and-timezone:latest .

For verification we run date in the container, which gives us zone CET :

$ docker run --rm ubuntu-locale-and-timezone:latest date 
Sun Mar 13 05:41:56 PM CET 2022
and localefor current locale and language settings:

$ docker run --rm ubuntu-locale-and-timezone:latest locale 
LANG=en_US.UTF-8 
LANGUAGE=en_US:en 
LC_CTYPE="en_US.UTF-8" 
LC_NUMERIC="en_US.UTF-8" 
LC_TIME="en_US.UTF-8" 
LC_COLLATE="en_US.UTF-8" 
LC_MONETARY="en_US.UTF-8" 
LC_MESSAGES="en_US.UTF-8" 
LC_PAPER="en_US.UTF-8" 
LC_NAME="en_US.UTF-8" 
LC_ADDRESS="en_US.UTF-8" 
LC_TELEPHONE="en_US.UTF-8" 
LC_MEASUREMENT="en_US.UTF-8" 
LC_IDENTIFICATION="en_US.UTF-8" 
LC_ALL=en_US.UTF-8


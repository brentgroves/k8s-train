# Set the timezone
# I tried many things to get tzdata to do something useful
# I think it has been depreciated in favor of systemd's 
# timedatectl program. Well everytime I install it the
# /etc/timezone link is changed to America/Adek so I 
# have decided not to install it 
# RUN apt-get update && \
#     apt-get install -yq tzdata && \
#     ln -fs /usr/share/zoneinfo/America/Fort_Wayne /etc/localtime && \
#     dpkg-reconfigure -f noninteractive tzdata

<!-- This is all that I do -->
# Set the timezone
ENV TZ=America/Fort_Wayne
RUN ln -fs /usr/share/zoneinfo/America/Fort_Wayne /etc/localtime && \
  echo $TZ | tee /etc/timezone 


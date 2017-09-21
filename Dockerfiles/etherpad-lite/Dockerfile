#Stable version of etherpad doesn't support npm 2
#run on Debian or Ubuntu.
#A Dockerfile has inspired by Tlacuaches Project
FROM debian:jessie
MAINTAINER Ismael Garcia <tuxisma@gmail.com>

ENV ETHERPAD_VERSION 1.6.1

RUN apt-get update && \
    apt-get install -y curl unzip nodejs-legacy npm mysql-client && \
    rm -r /var/lib/apt/lists/*

WORKDIR /opt/

RUN curl -SL \
    https://github.com/ether/etherpad-lite/archive/${ETHERPAD_VERSION}.zip \
    > etherpad.zip && unzip etherpad && rm etherpad.zip && \
    mv etherpad-lite-${ETHERPAD_VERSION} etherpad-lite

WORKDIR etherpad-lite

RUN bin/installDeps.sh && rm settings.json
COPY dbconfig.sh /dbconfig.sh

RUN sed -i 's/^node/exec\ node/' bin/run.sh

VOLUME /opt/etherpad-lite/var
RUN ln -s var/settings.json settings.json

EXPOSE 9001
ENTRYPOINT ["/dbconfig.sh"]
CMD ["bin/run.sh", "--root"]

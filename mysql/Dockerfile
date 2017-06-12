FROM alpine:latest
MAINTAINER BrokenStone  <wdmsyf@sina.com>

WORKDIR /app
VOLUME /app

RUN apk add --update mysql mysql-client && rm -f /var/cache/apk/*
COPY my.cnf /etc/mysql/my.cnf

COPY startup.sh /startup.sh
RUN chmod +x /startup.sh

COPY stop.sh /stop.sh
RUN chmod +x /stop.sh

EXPOSE 3306
CMD ["/startup.sh"]

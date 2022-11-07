FROM --platform=linux/x86_64 mysql:8.0

ENV TZ=UTC

COPY ./docker/dev/mysql8/my.cnf /etc/my.cnf
COPY ./docker/dev/mysql8/initdb.d/init.sql /docker-entrypoint-initdb.d

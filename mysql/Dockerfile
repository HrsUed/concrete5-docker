FROM mysql:latest

# For debugging
RUN apt-get update && apt-get install -y --no-install-recommends \
    iputils-ping \
    net-tools \
    procps \
  && rm -rf /var/lib/apt/lists/*

VOLUME /var/lib/mysql
# Config files
COPY config/ /etc/mysql/

COPY docker-entrypoint.sh /usr/local/bin/
RUN rm /entrypoint.sh \
    && ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]

# EXPOSE 3306 33060
CMD ["mysqld"]

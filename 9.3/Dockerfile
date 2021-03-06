FROM quay.io/aptible/debian:wheezy

# Define PostgreSQL version for shared scripts
ENV PG_VERSION 9.3

# cf. docker-library/postgres: explicitly create the user so uid and gid are consistent.
RUN groupadd -r postgres && useradd -r -g postgres postgres

# Temporary workaround for host-container user conflicts on Linux Kernel >= 3.15
# See https://github.com/docker/docker/issues/6345 for details.
RUN ln -s -f /bin/true /usr/bin/chfn

# Install PostgreSQL 9.3.x from official Debian sources
RUN apt-install locales wget unzip \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

ADD templates/etc /etc
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys \
      B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8 && \
    apt-get update && \
    apt-get -y install postgresql-common && \
    sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf && \
    apt-get -y install postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3 postgresql-9.3-postgis-2.1 && \
    rm -rf /var/lib/apt/lists/*

# Install self-signed certificate and disallow non-SSL connections
RUN mkdir -p /etc/postgresql/9.3/ssl && cd /etc/postgresql/9.3/ssl && \
    openssl req -new -newkey rsa:1024 -days 365000 -nodes -x509 \
      -keyout server.key -subj "/CN=PostgreSQL" -out server.crt && \
    chmod og-rwx server.key && chown -R postgres:postgres /etc/postgresql/9.3

ENV DATA_DIRECTORY /var/db
RUN mkdir $DATA_DIRECTORY && chown -R postgres $DATA_DIRECTORY

ADD test /tmp/test
RUN su postgres -c "/usr/lib/postgresql/9.3/bin/initdb -D "$DATA_DIRECTORY"" && \
    bats /tmp/test && \
    rm -rf "$DATA_DIRECTORY"/*

VOLUME ["$DATA_DIRECTORY"]
EXPOSE 5432

ADD run-database.sh /usr/bin/
ENTRYPOINT ["run-database.sh"]

ARG SO_BASE
FROM ${SO_BASE}

LABEL maintainer="Security Onion Solutions, LLC"
LABEL description="PostgreSQL running in Docker container for use with Security Onion"

# renovate: datasource=rpm depName=postgres packageName=postgresql17-server registryUrl=https://download.postgresql.org/pub/repos/yum/17/redhat/rhel-9-x86_64/repodata/ extractVersion=^(?<version>\d+\.\d+)-
ARG PG_VERSION=17.11
ENV PG_MAJOR=17 \
    PG_VERSION=${PG_VERSION} \
    PGDATA=/var/lib/postgresql/data \
    PATH=/usr/pgsql-17/bin:$PATH \
    LANG=en_US.utf8

HEALTHCHECK --interval=1m --timeout=10s CMD pg_isready -U postgres || exit 1
EXPOSE 5432

# PGDG repo and key, from pgdg-redhat-repo-latest.noarch.rpm
COPY files/pgdg/pgdg-redhat-all.repo /etc/yum.repos.d/
COPY files/pgdg/PGDG-RPM-GPG-KEY-RHEL /etc/pki/rpm-gpg/

# PostgreSQL 17 and extensions
RUN dnf install -y --nodocs shadow-utils && \
    dnf install -y --nodocs libicu python3-psycopg2 && \
    dnf install -y --disablerepo=* --enablerepo=pgdg17 \
        postgresql17-server-${PG_VERSION} \
        pgvector_17 \
        pg_cron_17 \
        pg_partman_17 && \
    dnf clean all && \
    rm -rf /var/cache/dnf/*

# gosu signing key: B42F 6819 007F 00F8 8E36 4FD4 036A 9C25 BF35 7DD4
COPY files/gosu.asc /tmp/gosu.asc
# renovate: datasource=github-releases depName=gosu packageName=tianon/gosu
ARG GOSU_VERSION=1.19
RUN curl -fsSL -o /usr/local/bin/gosu https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-amd64 && \
    curl -fsSL -o /tmp/gosu.sig https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-amd64.asc && \
    export GNUPGHOME="$(mktemp -d)" && \
    gpg --batch --quiet --import /tmp/gosu.asc && \
    gpg --batch --verify /tmp/gosu.sig /usr/local/bin/gosu && \
    rm -rf "$GNUPGHOME" /tmp/gosu.sig /tmp/gosu.asc && \
    chmod +x /usr/local/bin/gosu && \
    gosu --version

# Official entrypoint, docker-library/postgres@4f9ced003ba5
COPY --chmod=0755 files/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

RUN mkdir -p /log /conf "$PGDATA" /var/run/postgresql && \
    chown -R postgres:postgres /var/run/postgresql

RUN usermod -u 939 postgres && groupmod -g 939 postgres

COPY entrypoint.sh /entrypoint.sh
RUN chmod a+x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["postgres", "-c", "config_file=/conf/postgresql.conf"]

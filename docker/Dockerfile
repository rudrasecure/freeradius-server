# Set the base image and arguments
ARG from=alpine:3.13

# Build stage
FROM ${from} AS build

# Install build dependencies and clone the FreeRADIUS source
RUN apk update && apk add --no-cache git gcc make libc-dev talloc-dev openssl openssl-dev \
    linux-headers pcre-dev libidn-dev krb5-dev samba-dev curl-dev json-c-dev openldap-dev \
    unbound-dev ruby-dev perl-dev python2-dev hiredis-dev libmemcached-dev gdbm-dev \
    libcouchbase-dev postgresql-dev mariadb-dev unixodbc-dev sqlite-dev

WORKDIR /usr/local/src/repositories
RUN git clone --depth 1 --single-branch --branch rumars-modification https://github.com/rudrasecure/freeradius-server.git
WORKDIR /usr/local/src/repositories/freeradius-server

RUN ./configure --prefix=/opt && make -j12 && make install && rm /opt/lib/*.a

# Final stage
FROM ${from}

# Copy FreeRADIUS installation from the build stage
COPY --from=build /opt /opt

# Copy configuration files and entrypoint script
#COPY ./raddb/ /opt/etc/raddb/
COPY docker-entrypoint.sh /

# Install necessary runtime dependencies
RUN apk update && apk add --no-cache talloc libressl pcre libwbclient tzdata libcurl \
    json-c libldap hiredis sqlite-dev mariadb-dev

# Set working directory and create symlinks for SQL modules
WORKDIR /opt/etc/raddb/mods-enabled
RUN ln -s /opt/etc/raddb /etc/raddb && \
    ln -s ../mods-available/sql sql && \
    ln -s ../mods-available/sqlcounter sqlcounter && \
    chmod +x /docker-entrypoint.sh

# Expose RADIUS ports and set entrypoint
EXPOSE 1812/udp 1813/udp
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["radiusd"]

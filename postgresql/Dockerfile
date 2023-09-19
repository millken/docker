
FROM ubuntu:jammy AS add-apt-repositories

RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y wget gnupg \
 && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
 && echo 'deb http://apt.postgresql.org/pub/repos/apt/ jammy-pgdg main' >> /etc/apt/sources.list


FROM ubuntu:jammy  AS postgresql-dev

COPY --from=add-apt-repositories /etc/apt/trusted.gpg /etc/apt/trusted.gpg
COPY --from=add-apt-repositories /etc/apt/sources.list /etc/apt/sources.list

WORKDIR /app
ENV LANG en_US.utf8
ENV DEBIAN_FRONTEND noninteractive

ENV PG_MAJOR 15
ENV PG_IVM "1.7"
ENV PG_COLUMNAR "1.0.0-rc2"

RUN true \
    && apt-get -y install \
        -o APT::Install-Recommends="false" \
        -o APT::Install-Suggests="false" \
		libzstd-dev \
		libcurl4-openssl-dev \
		liblz4-dev \
		gcc \
		cmake \
        sudo \
        build-essential \
        curl \
        postgresql-$PG_MAJOR \
        postgresql-contrib-$PG_MAJOR \
		postgresql-client-$PG_MAJOR \
		postgresql-server-dev-$PG_MAJOR \

# Configure postgres.
RUN true \
    && echo "host all all 0.0.0.0/0 md5" >> /etc/postgresql/$PG_MAJOR/main/pg_hba.conf \
    && echo "listen_addresses='*'" >> /etc/postgresql/$PG_MAJOR/main/postgresql.conf

# pg_ivm extension install.
RUN true \
    && curl -L https://github.com/sraoss/pg_ivm/archive/refs/tags/v${PG_IVM}.tar.gz -o pg_ivm.tar.gz \
    && tar xf pg_ivm.tar.gz \
    && cd pg_ivm-${PG_IVM} \
    && make -j`nproc` \
    && DESTDIR=/app make install

# hydra columnar extension install.
RUN true \
    && curl -L curl -L https://github.com/hydradatabase/hydra/archive/refs/tags/v{PG_COLUMNAR}.tar.gz -o hydra.tar.gz \
    && tar xf hydra.tar.gz \
    && cd hydra-${PG_COLUMNAR}/columnar \
	&& ./configure \
	&& cd src/backend/columnar/ \
    && DESTDIR=/app make install

FROM ubuntu:jammy AS postgresql
LABEL maintainer="millken@gmail.com"
COPY --from=add-apt-repositories /etc/apt/trusted.gpg /etc/apt/trusted.gpg
COPY --from=add-apt-repositories /etc/apt/sources.list /etc/apt/sources.list

ENV PG_MAJOR 15
ENV LANG en_US.utf8
ENV DEBIAN_FRONTEND noninteractive

RUN true \
    && apt-get -y install \
        -o APT::Install-Recommends="false" \
        -o APT::Install-Suggests="false" \
        postgresql-$PG_MAJOR \
        postgresql-contrib-$PG_MAJOR 

# Configure postgres.
RUN true \
    && echo "host all all 0.0.0.0/0 md5" >> /etc/postgresql/$PG_MAJOR/main/pg_hba.conf \
    && echo "listen_addresses='*'" >> /etc/postgresql/$PG_MAJOR/main/postgresql.conf

FROM scratch

ENV LANG en_US.utf8
ENV DEBIAN_FRONTEND noninteractive

COPY --from=postgresql / /
COPY --from=postgresql-dev /app /
EXPOSE 5432

CMD ["/bin/sh"]

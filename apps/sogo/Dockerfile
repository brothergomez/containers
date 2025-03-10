# hadolint ignore=DL3007
FROM ghcr.io/truecharts/ubuntu:latest@sha256:f4b95e7f5a1c6c7736fef2a1800284058ccd53aa720536d0f1d72cf80faa4d2d AS builder

# Change your name and email address
ARG DEBFULLNAME=TrueCharts
ARG DEBEMAIL=info@truecharts.org

ARG TARGETPLATFORM
ARG VERSION


# hadolint ignore=DL3002
USER root

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /buildtmp
COPY ./apps/sogo/build.sh /buildtmp/build.sh

# hadolint ignore=DL3008,DL3015,SC2086,SC2155,DL3047
RUN \
  mkdir /build && \
  cd /buildtmp && \
  chmod 777 build.sh && \
  chmod +x build.sh && \
  ./build.sh


# hadolint ignore=DL3007
FROM ghcr.io/truecharts/ubuntu:latest@sha256:f4b95e7f5a1c6c7736fef2a1800284058ccd53aa720536d0f1d72cf80faa4d2d

ARG TARGETPLATFORM
ARG VERSION

ARG DEBIAN_FRONTEND=noninteractive
ARG SOGO_UBUNTU_REPOSITORY=https://packages.inverse.ca/SOGo/nightly/5/ubuntu/
ENV LC_ALL C
ENV GOSU_VERSION 1.12

# hadolint ignore=DL3002
USER root

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008,DL3015,SC2086,SC2155,DL3047
RUN --mount=type=bind,from=builder,source=/buildtmp/vendor/output,target=/build \
  apt-get -qq update \
  && \
  apt-get -qq install -y \
    libicu66 \
    jq \
    apt-transport-https \
    ca-certificates \
    gettext \
    gnupg \
    mariadb-client \
    libmariadb3 \
    libcurl3-gnutls \
    rsync \
    supervisor \
    syslog-ng \
    syslog-ng-core \
    syslog-ng-mod-redis \
    dirmngr \
    netcat \
    psmisc \
    wget \
    patch \
    apache2 \
    gnupg2 \
    memcached \
    postgresql-client \
  && \
  case "${TARGETPLATFORM}" in \
    'linux/amd64') export ARCH='linux-x64' ;; \
  esac \
  && \
  mkdir ~/.gnupg && \
  echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf && \
  mkdir /usr/share/doc/sogo && \
  touch /usr/share/doc/sogo/empty.sh && \
  cd /build && \
  (echo "$(dpkg-deb -I _sogo_${VERSION}_amd64.deb | grep -E "Depends" | tr -d "|," | sed "s/([^)]*)/()/g" | tr -d "()" | tr " " "\n" | grep -Ev "Depends|Recommends|Suggests|Pre\-Depends|sogo|sope" | xargs)" | xargs -n 1 apt-get install -y || true) && \
  apt install -y ./*.deb && \
  apt-get install -f && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
  a2enmod headers proxy proxy_http rewrite ssl && \
  cp /etc/apache2/conf.d/SOGo.conf /etc/apache2/conf-available/ && \
  sed -i -e 's/#RedirectMatch \^\/\$ https:\/\/mail.yourdomain.com\/SOGo/RedirectMatch \^\/\$ \/SOGo/' /etc/apache2/conf-available/SOGo.conf && \
  sed -i -e 's|"x-webobjects-server-url" "https://|"x-webobjects-server-url" "http://|' /etc/apache2/conf-available/SOGo.conf && \
  sed -i -e 's|"x-webobjects-server-port" "443"|"x-webobjects-server-port" "80"|' /etc/apache2/conf-available/SOGo.conf && \
  a2enconf SOGo && \
  usermod --home /srv/lib/sogo sogo && \
  mkdir -p /var/run/memcached/ && \
    chown memcache:memcache /var/run/memcached

EXPOSE 80 443

COPY ./apps/sogo/etc /etc
COPY ./apps/sogo/entrypoint.sh /entrypoint.sh


CMD ["supervisord", "-c", "/etc/supervisor/supervisord.conf"]

LABEL "maintainer"="TrueCharts <info@truecharts.org>"
LABEL "org.opencontainers.image.source"="https://github.com/truecharts/apps"

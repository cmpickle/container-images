# hadolint ignore=DL3007
FROM ghcr.io/k8s-at-home/ubuntu:latest as builder

ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG VERSION

# hadolint ignore=DL3002
USER root

WORKDIR /tmp

# Install system dependencies
# hadolint ignore=DL3008
RUN \
  apt-get -qq update \
  && \
  apt-get -qq install -y --no-install-recommends \
    autoconf \
    automake \
    build-essential \
    cmake \
    git \
    intltool \
    libappindicator3-dev \
    libcurl4-openssl-dev \
    libevent-dev \
    libglib2.0-dev \
    libminiupnpc-dev \
    libssh-dev \
    libtool \
    pkg-config \
    software-properties-common \
    gpg \
    dirmngr

WORKDIR /tmp/transmission

# Compile transmission
RUN \
  git clone --depth 1 -b ${VERSION} https://github.com/transmission/transmission.git . \
  && git submodule update --init \
  && ./autogen.sh --without-gtk \
  && make -s \
  && make install

# hadolint ignore=DL3007
FROM ghcr.io/k8s-at-home/ubuntu:latest

ARG TARGETPLATFORM
ARG VERSION

# Proper way to set config directory
ENV HOME=/config \
    XDG_CONFIG_HOME=/config \
    XDG_DATA_HOME=/config \
    WEBUI_PORT=9091 \
    TRANSMISSION_WEB_HOME=/web

USER root

COPY --from=builder /usr/local/bin/transmission-daemon /app/transmission-daemon
COPY --from=builder /usr/local/share/transmission/web /web

# hadolint ignore=DL3008,DL3015
RUN \
  apt-get -qq update \
  && \
  apt-get install -y \
    libevent-dev \
    libminiupnpc-dev \
    p7zip-full \
    python3 \
    rsync \
    unrar \
    unzip \
  && \
  apt-get autoremove -y \
  && apt-get clean \
  && \
  rm -rf \
    /tmp/* \
    /var/lib/apt/lists/* \
    /var/tmp/ \
  && chown -R kah:kah /app \
  && chown -R kah:kah /web \
  && chmod -R u=rwX,go=rX /app \
  && chmod -R u=rwX,go=rX /web \
  && printf "umask %d" "${UMASK}" >> /etc/bash.bashrc \
  && update-ca-certificates

USER kah

EXPOSE ${WEBUI_PORT} 51413

COPY ./apps/transmission/settings.json.tmpl /app/settings.json.tmpl
COPY ./apps/transmission/entrypoint.sh /entrypoint.sh
CMD ["/entrypoint.sh"]

LABEL \
  org.opencontainers.image.title="Transmission" \
  org.opencontainers.image.source="https://github.com/transmission/transmission" \
  org.opencontainers.image.version="${VERSION}"

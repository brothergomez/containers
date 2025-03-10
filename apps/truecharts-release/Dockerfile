# hadolint ignore=DL3007
FROM quay.io/helmpack/chart-releaser:v1.3.0@sha256:be34d8dbf00e44bf52fbade53f7e3f96277581a815628e23817b23827347ade1 AS cr

# hadolint ignore=DL3007
FROM jnorwood/helm-docs:v1.7.0@sha256:a1405b3c86c48f4178543ed03d40f7c786228acb48a4ee85b670803cf1effe78 AS hd

# hadolint ignore=DL3007
FROM quay.io/git-chglog/git-chglog:0.15.1@sha256:471db4cefc1a4dc0d8ee996447d5ac860f9496c982bcd29e90b390aff4e78175 AS gcl

# hadolint ignore=DL3007
FROM ixsystems/catalog_validation:latest@sha256:64e021e40a94356bc3146112b2b8a4efe9ca9a861ada4803b659aad462fb14ea

ARG TARGETPLATFORM
ARG VERSION

USER root

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008,DL3015,SC2086,SC2155
RUN --mount=type=bind,from=cr,source=/usr/local/bin,target=/crbin \
  cp /crbin/cr /usr/local/bin/cr

# hadolint ignore=DL3008,DL3015,SC2086,SC2155
RUN --mount=type=bind,from=hd,source=/usr/bin/,target=/hdbin \
  cp /hdbin/helm-docs /usr/local/bin/helm-docs

# hadolint ignore=DL3008,DL3015,SC2086,SC2155
RUN --mount=type=bind,from=gcl,source=/usr/local/bin,target=/gclbin \
  cp /gclbin/git-chglog /usr/local/bin/git-chglog

COPY ./.pre-commit-config.yaml /app/.pre-commit-config.yaml

# hadolint ignore=DL3008,DL3015,SC2086,SC2155,DL3013
RUN pip install --no-cache-dir pre-commit && \
  git init . && \
  pre-commit install-hooks && \
  rm -Rf /App/* && \
  echo "installed pre-commit hooks:" && \
  ls ~/.cache/pre-commit/*


LABEL "maintainer"="TrueCharts <info@truecharts.org>"
LABEL "org.opencontainers.image.source"="https://github.com/truecharts/apps"

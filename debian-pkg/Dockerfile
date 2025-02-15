# syntax=docker/dockerfile:1.4
# Enable here-documents:
# https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#here-documents

FROM debian:bullseye-20220328-slim AS build

RUN set -x && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
      debhelper \
      dpkg-dev

# The canonical, SemVer-compliant TinyPilot version.
ARG TINYPILOT_VERSION

# The `PKG_VERSION` is the version of the Debian package. Debian uses its own
# versioning scheme, which is not strictly compatible with TinyPilot’s version
# conventions. Therefore, the package version should be a timestamp, formatted
# `YYYYMMDDhhmmss`. That way the package manager always installs the most
# recently built TinyPilot package.
ARG PKG_VERSION

ARG PKG_NAME="tinypilot"
ARG PKG_BUILD_NUMBER="1"
ARG PKG_ARCH="all"
ARG PKG_ID="${PKG_NAME}-${PKG_VERSION}-${PKG_BUILD_NUMBER}-${PKG_ARCH}"

RUN mkdir -p "/releases/${PKG_ID}"

WORKDIR "/releases/${PKG_ID}"

COPY ./debian-pkg ./
COPY ./COPYRIGHT ./
COPY ./LICENSE ./
COPY ./README.md ./
COPY ./requirements.txt ./
COPY ./app ./app
COPY ./scripts ./scripts

RUN echo "${TINYPILOT_VERSION}" > VERSION

WORKDIR "/releases/${PKG_ID}/debian"

RUN cat >control <<EOF
Source: ${PKG_NAME}
Section: net
Priority: optional
Maintainer: TinyPilot Support <support@tinypilotkvm.com>
Build-Depends: debhelper (>= 11)

Package: ${PKG_NAME}
Architecture: ${PKG_ARCH}
Depends: adduser, python3, python3-pip, python3-venv, sudo
Homepage: https://tinypilotkvm.com
Description: Simple, easy-to-use KVM over IP
XBS-Tinypilot-Version: ${TINYPILOT_VERSION}
EOF

RUN cat >changelog <<EOF
tinypilot (${PKG_VERSION}) bullseye; urgency=medium

  * Latest TinyPilot release.

 -- TinyPilot Support <support@tinypilotkvm.com>  $(date '+%a, %d %b %Y %H:%M:%S %z')
EOF

WORKDIR "/releases/${PKG_ID}"
RUN dpkg-buildpackage --build=binary

FROM scratch as artifact

COPY --from=build "/releases/*.deb" ./

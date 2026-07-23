FROM govpf/debian:bookworm
 
RUN groupadd --gid 1000 node \
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node
 
RUN apt-get update && \
    DEBIAN_FRONTEND="noninteractive" \
    apt-get install -y --no-install-recommends \
      xz-utils \
    && rm -rf /var/lib/apt/lists/*
 
ENV NODE_VERSION=24.18.0
 
RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
  && case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
  esac \
  && set -ex \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt.asc | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
  && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
  && node --version \
  && npm --version
 
# Yarn 1.x n'est pas installe dans cette image.
# La procedure amont recupere la clef de signature Yarn sur un serveur de clefs
# public au moment du build ; les deux serveurs utilises sont inexploitables
# depuis le reseau DSI :
#   - keyserver.ubuntu.com : contacte en hkp:// (port 11371), filtre -> timeout
#   - keys.openpgp.org     : renvoie la clef sans UID, refusee par GnuPG
# npm (fourni avec Node) couvre les besoins des projets Angular. Si un projet
# exige Yarn, l'installer au niveau du projet plutot que dans l'image de base.
 
ENV NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt
 
COPY 24/debian/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
 
CMD [ "node" ]
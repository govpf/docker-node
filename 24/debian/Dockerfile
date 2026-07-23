
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
 
# Node 24.18.0 embarque npm 11.16.0 ; on monte a la derniere majeure.
# engines npm@12.0.1 : node ^22.22.2 || ^24.15.0 || >=26.0.0 -> OK avec 24.18.0
ENV NPM_VERSION=12.0.1
 
RUN npm install -g "npm@$NPM_VERSION" \
  && npm cache clean --force \
  && npm --version \
  && rm -rf /tmp/*
 
ENV YARN_VERSION=1.22.22
 
# Yarn 1.x est installe depuis le registre npm, et non depuis yarnpkg.com :
# npm verifie l'integrite du paquet via le SHA-512 publie par le registre, sans
# dependre d'un serveur de clefs GPG (keyserver.ubuntu.com est filtre en hkp://
# port 11371, et keys.openpgp.org renvoie la clef sans UID -> refusee par GnuPG).
# Le paquet expose les binaires yarn et yarnpkg, comme la procedure amont.
RUN npm install -g "yarn@$YARN_VERSION" \
  && npm cache clean --force \
  && yarn --version \
  && rm -rf /tmp/*
 
ENV NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt
 
COPY 24/debian/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
 
CMD [ "node" ]
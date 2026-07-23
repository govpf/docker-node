FROM govpf/alpine:3.24
 
RUN apk upgrade && \
    apk add --no-cache xz
 
ENV NODE_VERSION=24.18.0
 
RUN addgroup -g 1000 node \
    && adduser -u 1000 -G node -s /bin/sh -D node \
    && apk add --no-cache \
        libstdc++ \
    && apk add --no-cache --virtual .build-deps \
        curl \
    && ARCH= OPENSSL_ARCH='linux*' && alpineArch="$(apk --print-arch)" \
      && case "${alpineArch##*-}" in \
        x86_64) ARCH='x64' CHECKSUM="b818a0c3857272329cad4d575abf49e5060215858c9c3015437366f8adc7b85d" OPENSSL_ARCH=linux-x86_64;; \
        x86) OPENSSL_ARCH=linux-elf;; \
        aarch64) OPENSSL_ARCH=linux-aarch64;; \
        arm*) OPENSSL_ARCH=linux-armv4;; \
        ppc64le) OPENSSL_ARCH=linux-ppc64le;; \
        s390x) OPENSSL_ARCH=linux-s390x;; \
        *) ;; \
      esac \
  # x86_64 : binaire musl pre-compile, verifie par SHA256 (~1 min de build).
  # Autres archs : compilation depuis les sources, verifiee par GPG (long).
  && if [ -n "${CHECKSUM}" ]; then \
    set -eu; \
    curl -fsSLO --compressed "https://unofficial-builds.nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz"; \
    echo "$CHECKSUM  node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" | sha256sum -c - \
      && tar -xJf "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
      && ln -s /usr/local/bin/node /usr/local/bin/nodejs; \
  else \
    echo "Building from source" \
    # backup build
    && apk add --no-cache --virtual .build-deps-full \
        binutils-gold \
        g++ \
        gcc \
        gnupg \
        libgcc \
        linux-headers \
        make \
        python3 \
        py-setuptools \
    # use pre-existing gpg directory, see https://github.com/nodejs/docker-node/pull/1895#issuecomment-1550389150
    && export GNUPGHOME="$(mktemp -d)" \
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    # (liste mise a jour pour la branche 24 : 4 anciennes clefs retirees, 1 ajoutee)
    # hkps:// impose le port 443 ; hkp:// (port 11371) est filtre en reseau DSI.
    # Le --fingerprint fait echouer le build si la clef n'a pas ete importee.
    && for key in \
      5BE8A3F6C8A5C01D106C0AD820B1A390B168D356 \
      DD792F5973C6DE52C432CBDAC77ABFA00DDBF2B7 \
      CC68F5A3106FF448322E48ED27F5E38D5B0A215F \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      890C08DB8579162FEE0DF9DB8BEAB4DFCF555EF4 \
      C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
      108F52B48DB57BB0CC439B2997B01419BD92F80A \
      A363A499291CBBC940DD62E41F10027AF002F8B0 \
    ; do \
      { gpg --batch --keyserver hkps://keys.openpgp.org --keyserver-options timeout=15 --recv-keys "$key" && gpg --batch --fingerprint "$key"; } || \
      { gpg --batch --keyserver hkps://keyserver.ubuntu.com --keyserver-options timeout=15 --recv-keys "$key" && gpg --batch --fingerprint "$key"; } ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xf "node-v$NODE_VERSION.tar.xz" \
    && cd "node-v$NODE_VERSION" \
    && ./configure \
    # Chaque job de compilation de V8 peut atteindre ~2 Go de RAM : un -j non
    # borne (1 job par coeur) fait tomber la machine en OOM. On plafonne.
    && MAKE_JOBS=$(awk -v c="$(getconf _NPROCESSORS_ONLN)" \
         '/MemTotal/ {m=int($2/1024/1024/2); if (m<1) m=1; print (m<c ? m : c)}' /proc/meminfo) \
    && echo "make -j$MAKE_JOBS" \
    && make -j"$MAKE_JOBS" V= \
    && make install \
    && apk del .build-deps-full \
    && cd .. \
    && rm -Rf "node-v$NODE_VERSION" \
    && rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt; \
  fi \
  && rm -f "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" \
  # Remove unused OpenSSL headers to save ~34MB. See this NodeJS issue: https://github.com/nodejs/node/issues/46451
  && find /usr/local/include/node/openssl/archs -mindepth 1 -maxdepth 1 ! -name "$OPENSSL_ARCH" -exec rm -rf {} \; \
  && apk del .build-deps \
  # smoke tests
  && node --version \
  && npm --version \
  && rm -rf /tmp/*
 
# Yarn 1.x n'est pas installe dans cette image.
# La procedure amont recupere la clef de signature Yarn sur un serveur de clefs
# public au moment du build ; les deux serveurs utilises sont inexploitables
# depuis le reseau DSI :
#   - keyserver.ubuntu.com : contacte en hkp:// (port 11371), filtre -> timeout
#   - keys.openpgp.org     : renvoie la clef sans UID, refusee par GnuPG
# npm (fourni avec Node) couvre les besoins des projets Angular. Si un projet
# exige Yarn, l'installer au niveau du projet plutot que dans l'image de base.
 
COPY 24/alpine/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
 
CMD [ "node" ]
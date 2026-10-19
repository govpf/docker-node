FROM govpf/alpine:3.24

LABEL govpf.fromimage="govpf/alpine:3.24"
LABEL govpf.category="runtime"

WORKDIR /app

RUN apk add --no-cache nodejs npm yarn tini openssl curl bash ca-certificates \
    && apk upgrade --no-cache \
    && npm install -g npm@8.19.4 \
    && npm install sharp \
    && curl -fsSL https://raw.githubusercontent.com/govpf/repo-utils/master/scripts/docker/install_certs | bash -

WORKDIR /

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["bash"]
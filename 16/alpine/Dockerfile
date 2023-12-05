FROM govpf/alpine:3

WORKDIR /app

RUN apk update \
    && apk add nodejs npm yarn tini \
    && apk upgrade --no-cache \
    && npm install -g npm@8.19.4 \
    && npm install sharp \
    && curl -sL https://raw.githubusercontent.com/govpf/repo-utils/master/scripts/docker/install_certs | bash -
    
WORKDIR /

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["bash"]

FROM govpf/alpine:3

WORKDIR /app

RUN apk update \
    && apk add nodejs=16.19.1-r0 npm=8.1.3-r0 yarn=1.22.17-r0 tini \
    && apk upgrade --no-cache \
    && npm i -g npm \
    && npm install --platform=linuxmusl --arch=x64 sharp \
    && curl -sL https://raw.githubusercontent.com/govpf/repo-utils/master/scripts/docker/install_certs | bash -

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["bash"]

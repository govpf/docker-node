FROM govpf/alpine:3

RUN apk update \
    && apk add nodejs=16.19.1-r0 npm=8.1.3-r0 yarn=1.22.17-r0 tini \
    && apk upgrade --no-cache \
    && npm i -g npm

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["bash"]

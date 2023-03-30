FROM govpf/debian:bullseye

RUN curl -sL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get update \
    && apt-get install -y --no-install-recommends nodejs yarn dumb-init \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && rm -rf /var/lib/apt/lists/* \
    && npm i -g npm \
    && curl -sL https://raw.githubusercontent.com/govpf/repo-utils/master/scripts/docker/install_certs | bash -

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["bash"]

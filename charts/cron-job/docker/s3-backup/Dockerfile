FROM amazon/aws-cli:2.33.17

# Use root only when necessary, minimize layers, and clean up after install
USER root

RUN set -eux; \
    yum install -y tar gzip; \
    yum clean all; \
    rm -rf /var/cache/yum

# Drop privileges after installing packages for better security
USER 1000:1000

# ENTRYPOINT should not be empty unless overridden; keep only CMD unless a custom entrypoint is required
CMD ["/bin/bash"]

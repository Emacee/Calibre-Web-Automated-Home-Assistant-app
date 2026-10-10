# Calibre-Web Automated as a Home Assistant add-on
#
# The upstream image declares VOLUME /config, /calibre-library and
# /cwa-book-ingest. Under the HA Supervisor those would become anonymous
# Docker volumes that are thrown away whenever the add-on container is
# recreated (restart with changed options, update), taking the library with
# them. A Dockerfile cannot remove a VOLUME, so stage 2 flattens the image
# into a fresh one without them; the add-on keeps its data in /config
# (addon_config) and /share or /media instead.
ARG CWA_VERSION=v4.0.8

FROM crocodilestick/calibre-web-automated:${CWA_VERSION} AS cwa

# nginx: ingress proxy (X-Ingress-Path -> X-Script-Name)
RUN apt-get update \
    && apt-get install -y --no-install-recommends nginx \
    && rm -rf /var/lib/apt/lists/* /etc/nginx/sites-enabled/*

COPY rootfs/ /
RUN chmod +x /etc/s6-overlay/s6-rc.d/init-ha-addon/run \
             /etc/s6-overlay/s6-rc.d/svc-ha-ingress/run \
             /etc/s6-overlay/s6-rc.d/init-ha-library-perms/run

FROM scratch
COPY --from=cwa / /

# Config of the upstream image (lost by flattening)
ENV PATH="/lsiopy/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    HOME="/root" \
    LANGUAGE="en_US.UTF-8" \
    LANG="en_US.UTF-8" \
    TERM="xterm" \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME="0" \
    S6_VERBOSITY="1" \
    S6_STAGE2_HOOK="/docker-mods" \
    VIRTUAL_ENV="/lsiopy" \
    CALIBRE_CONFIG_DIR="/config/.config/calibre"

# Stop the add-on (visible in HA) when an init step such as the option check
# fails, instead of running on half-initialised
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS="2"

LABEL io.hass.type="addon" \
      io.hass.arch="aarch64|amd64"

WORKDIR /config
EXPOSE 8083 8099
ENTRYPOINT ["/init"]

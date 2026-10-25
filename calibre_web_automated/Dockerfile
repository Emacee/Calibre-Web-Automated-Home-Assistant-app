# Calibre-Web Automated as a Home Assistant app
#
# The upstream image declares VOLUME /config, /calibre-library and
# /cwa-book-ingest. Under the HA Supervisor those would become anonymous
# Docker volumes that are thrown away whenever the app container is
# recreated (restart with changed options, update), taking the library with
# them. A Dockerfile cannot remove a VOLUME, so stage 2 flattens the image
# into a fresh one without them; the app keeps its data in /config
# (app_config) and /share or /media instead.
ARG CWA_VERSION=v4.0.8

FROM crocodilestick/calibre-web-automated:${CWA_VERSION} AS cwa

# nginx: ingress proxy (X-Ingress-Path -> X-Script-Name)
# apt-get upgrade: the upstream image is only as current as its release, so
# this applies the Ubuntu security fixes published since (ImageMagick and
# Ghostscript, which open every uploaded book and cover, among them) each
# time the app is built.
RUN apt-get update \
    && apt-get upgrade -y --no-install-recommends \
    && apt-get install -y --no-install-recommends nginx \
    && rm -rf /var/lib/apt/lists/* /etc/nginx/sites-enabled/*

# Calibre ships in /app/calibre, but upstream runs its post-install (command
# links in /usr/bin and such) at every container start, which takes minutes
# on a Home Assistant machine. Do it once here; CWA's calibre-binaries-setup
# then finds calibredb and skips it.
RUN /app/calibre/calibre_postinstall \
    && calibredb --version

COPY rootfs/ /
RUN chmod +x /etc/s6-overlay/s6-rc.d/init-ha-addon/run \
             /etc/s6-overlay/s6-rc.d/svc-ha-ingress/run \
             /etc/s6-overlay/s6-rc.d/init-ha-library-perms/run \
             /etc/s6-overlay/s6-rc.d/svc-ha-ingress-login/run \
             /usr/local/bin/ha-ingress-login

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

# Stop the app (visible in HA) when an init step such as the option check
# fails, instead of running on half-initialised
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS="2"

WORKDIR /config
EXPOSE 8084 8099
ENTRYPOINT ["/init"]

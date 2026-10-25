# Shelfmark as a Home Assistant app
#
# The upstream image runs dumb-init -> /app/entrypoint.sh. A wrapper is put in
# between that turns the app options (and what the Calibre-Web Automated app
# shares) into Shelfmark's environment variables and starts an nginx proxy
# for Home Assistant Ingress.
ARG SHELFMARK_VERSION=v1.5.1

FROM ghcr.io/calibrain/shelfmark:${SHELFMARK_VERSION}

# nginx: ingress proxy (X-Ingress-Path -> <base href>); jq: options.json
# apt-get upgrade: applies the Debian security fixes published since the
# upstream image was built. Chromium stays at the version upstream pins
# (apt preference with priority 1001), so its bypasser keeps working.
RUN apt-get update \
    && apt-get upgrade -y --no-install-recommends \
    && apt-get install -y --no-install-recommends nginx jq \
    && rm -rf /var/lib/apt/lists/* /etc/nginx/sites-enabled/*

COPY rootfs/ /
RUN chmod +x /usr/local/bin/ha-run /usr/local/bin/ha-cwa-token \
    /usr/local/bin/ha-want-to-read

EXPOSE 8084 8099
ENTRYPOINT ["/usr/bin/dumb-init", "--", "/usr/local/bin/ha-run"]
CMD ["/app/entrypoint.sh"]

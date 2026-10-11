# Changelog

## 4.0.12

Still based on Calibre-Web Automated v4.0.8.

- Share the user of the automatic Ingress login (name and admin role, not
  the header) with the Shelfmark app, so it logs in as the same user

## 4.0.11

Still based on Calibre-Web Automated v4.0.8.

- Log in automatically as the CWA admin (or `ingress_user`) when the app is
  opened from Home Assistant, via Calibre-Web's reverse proxy login with a
  random header name per installation; the direct port 8083 still asks for a
  password. New options `ingress_auto_login` (default on) and `ingress_user`

## 4.0.10

Still based on Calibre-Web Automated v4.0.8.

- Share the library and ingest folder, and the Hardcover token, with the new
  Shelfmark app from this repository, via `.ha-share` in the app config
  folder (token file only readable by the app itself)

## 4.0.9

Still based on Calibre-Web Automated v4.0.8.

- Use Home Assistant's current app conventions: map type `app_config`
  instead of the legacy `addon_config` (same folder, still mounted at
  `/config`), no hardcoded `io.hass.*` image labels
- Documentation uses the new "apps" naming

## 4.0.8

- Initial release, based on Calibre-Web Automated v4.0.8
- Home Assistant Ingress via an nginx proxy that maps `X-Ingress-Path` to
  Calibre-Web's `X-Script-Name` subpath support
- Library and ingest folder configurable under `/share` or `/media`;
  settings and users in the app config folder, so all survive updates
- Options for Hardcover token, network share mode, ingest watch mode and
  extra environment variables

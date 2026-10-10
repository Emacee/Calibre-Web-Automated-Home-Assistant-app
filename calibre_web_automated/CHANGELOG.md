# Changelog

## 4.0.8

- Initial release, based on Calibre-Web Automated v4.0.8
- Home Assistant Ingress via an nginx proxy that maps `X-Ingress-Path` to
  Calibre-Web's `X-Script-Name` subpath support
- Library and ingest folder configurable under `/share` or `/media`;
  settings and users in the app config folder, so all survive updates
- Options for Hardcover token, network share mode, ingest watch mode and
  extra environment variables

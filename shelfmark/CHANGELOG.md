# Changelog

## 1.4.0

- Initial release, based on Shelfmark v1.4.0
- Home Assistant Ingress via an nginx proxy that puts the ingress path in
  Shelfmark's `<base href>`
- Takes the ingest folder, Calibre library, Hardcover token and user
  database from the Calibre-Web Automated app, so it works without setup
- Options to override the download folder and Hardcover token, to turn off
  the Calibre-Web Automated login, and for extra environment variables

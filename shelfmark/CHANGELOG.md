# Changelog

## 1.4.0.3

Still based on Shelfmark v1.4.0.

- Take the Hardcover token from wherever it is set in Calibre-Web Automated,
  on every start: the CWA profile of the Ingress user (or the first admin
  with one), the CWA app options, or CWA's Basic Configuration. CWA's user
  database is read from a private snapshot, so recent changes count too

## 1.4.0.2

Still based on Shelfmark v1.4.0.

- Log in automatically when opened from Home Assistant, as the same user as
  the Calibre-Web Automated app (with its admin role), via Shelfmark's proxy
  login with a random header name per installation. New option
  `ingress_auto_login` (default on); needs Calibre-Web Automated 4.0.12

## 1.4.0.1

Still based on Shelfmark v1.4.0.

- Downloads always go to the ingest folder of the Calibre-Web Automated app;
  the `ingest_path` option is removed
- Lock the Shelfmark settings that would bypass or break the CWA import:
  output mode folder, books and audiobooks to the ingest folder (also for
  per-user destinations), rename without folders, no hardlinks, and only
  formats CWA can import
- **Go To Library** button opens the Calibre-Web Automated app

## 1.4.0

- Initial release, based on Shelfmark v1.4.0
- Home Assistant Ingress via an nginx proxy that puts the ingress path in
  Shelfmark's `<base href>`
- Takes the ingest folder, Calibre library, Hardcover token and user
  database from the Calibre-Web Automated app, so it works without setup
- Options to override the download folder and Hardcover token, to turn off
  the Calibre-Web Automated login, and for extra environment variables

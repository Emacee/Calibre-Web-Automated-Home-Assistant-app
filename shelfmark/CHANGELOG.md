# Changelog

## 1.5.1

Based on Shelfmark v1.5.1 (was v1.4.0). Mainly security and data-safety
fixes upstream:

- Only an admin may use the Hardcover account's lists and shelves (add or
  remove books, change read status, the List filter). Home Assistant users
  without the admin role in Calibre-Web Automated no longer see them. The
  Want to Read watcher is unaffected: Shelfmark treats its API key as admin
- Secrets, file links and server paths kept out of logs, error responses
  and status messages of non-admins
- Users can no longer see or change each other's activity items; the
  request policy now also applies to listing releases; cover cache keyed by
  image URL instead of a caller-supplied id
- Soulseek (slskd): unsafe peer paths are refused
- Downloads: no longer deletes user data or loses completed downloads
- Tor runs as `debian-tor` instead of root and survives container restarts
- Newer Python (3.14.8), Chromium (155) and dependencies (oauthlib 4.0.0);
  no C compiler and no pip in the image anymore

## 1.4.0.8

Still based on Shelfmark v1.4.0.

- Books on your Hardcover **Want to Read** list that are in your Calibre
  library (downloaded and imported) are moved to **Currently Reading**,
  checked every 15 minutes. Want to Read is now "still to download",
  Currently Reading "in the library". New option
  `want_to_read_move_when_imported` (default on); needs the `write:library`
  scope on the Hardcover token

## 1.4.0.7

Still based on Shelfmark v1.4.0.

- Download the books on your Hardcover **Want to Read** list automatically.
  Shelfmark itself only searches when you search; a watcher in this app now
  checks the list every few hours (new options `want_to_read_auto_download`,
  default on, and `want_to_read_interval_hours`, default 6), skips books
  already in your Calibre library and queues the best EPUB (or other format
  CWA imports) through Shelfmark, so it shows up under Activity

## 1.4.0.6

Still based on Shelfmark v1.4.0.

- New option `tor` (default off) to route all of Shelfmark's traffic through
  Tor, using the Tor support already in the Shelfmark image. The app now has
  the `NET_ADMIN` and `NET_RAW` permissions this needs, and a watchdog
  (TCP on the Ingress port) so Home Assistant can restart it when it stops

## 1.4.0.5

Still based on Shelfmark v1.4.0.

- Log Hardcover's own reason when it refuses a token, and still use a token
  that can search but misses the account scopes (with a warning naming the
  scopes to add)

## 1.4.0.4

Still based on Shelfmark v1.4.0.

- Check every Hardcover token found with Hardcover at startup and use the
  first one it accepts, so an old or unscoped token (HTTP 403) no longer
  blocks a working one; the log names the account and explains refused
  tokens, including the scopes a new token needs

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

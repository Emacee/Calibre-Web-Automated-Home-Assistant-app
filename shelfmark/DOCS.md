# Shelfmark — Documentation

This app runs [Shelfmark](https://github.com/calibrain/shelfmark) on your Home
Assistant machine. Shelfmark searches for books, including the books on your
Hardcover lists such as **Want to Read**, and downloads them. Downloads are
saved in the ingest folder of the **Calibre-Web Automated** app from this
repository, which imports them into your library. From there they reach your
e-reader, for example through Kobo sync.

It works together with the Calibre-Web Automated app and takes its settings
from there, so normally you don't have to configure anything.

## First start

1. Make sure the **Calibre-Web Automated** app is updated to 4.0.12 or later
   and has been started once since the update. That start is when it shares
   its settings.
2. Optional: set your **Hardcover API token** in Calibre-Web Automated, for
   example in your CWA profile (get one at
   [hardcover.app/account/api](https://hardcover.app/account/api)). See
   "Hardcover token" below for all places it is taken from.
3. Install and start this app, then click **Open web UI**. Turn on **Show in
   sidebar** to get a **Shelfmark** entry in the sidebar.
4. You are logged in automatically as the same user as in the Calibre-Web
   Automated app (see below).
5. On first use Shelfmark shows a short setup wizard to choose where books
   are searched for.

## What is taken from Calibre-Web Automated

| What | Used for | Override in this app |
|---|---|---|
| Ingest folder | Where every download is saved, so CWA imports it | — |
| Calibre library | Marks books you already own in search results | — |
| The app itself | **Go To Library** button, opens CWA in a new tab | — |
| Hardcover API token (see below) | Hardcover search and your Hardcover lists | `hardcover_token` |
| The Ingress user and its admin role | Automatic login from Home Assistant | `ingress_auto_login: false` |
| User accounts (`app.db`) | Password login when not logged in automatically | `use_cwa_login: false` |

Settings set this way are locked in Shelfmark's own settings screen. Restart
this app after changing a CWA password, or after changing the Hardcover
token or folders in the Calibre-Web Automated app (restart that app first).

## Hardcover token

On every start this app looks for a Hardcover token in this order and uses
the first one it finds:

1. `hardcover_token` in this app's options
2. The token in the **CWA profile** of the user you are logged in as through
   Ingress (or else of the first CWA admin that has one). Hardcover lists
   such as Want to Read are personal, so this is usually the right account.
3. `hardcover_token` in the Calibre-Web Automated app's options
4. The token in CWA under **Admin → Basic Configuration**

Each token found is checked with Hardcover at startup, and the first one
Hardcover accepts is used. The log shows where it came from and which
Hardcover account it belongs to (never the token itself), for example
`Hardcover token: CWA profile of admin, connected as <username>`. A token
Hardcover refuses is skipped, with the reason in the log. After changing the
token in CWA, restart this app. Shelfmark's own Hardcover settings then show
it as set by the environment (**ENV**).

### Making a token that works

Since August 2026 Hardcover issues tokens starting with `hc_pat_`, with
permissions (scopes). Older tokens (long ones starting with `eyJ`) no longer
work, and a token without the right scopes is refused with **HTTP 403**.
Create a new one at [hardcover.app/account/api](https://hardcover.app/account/api)
with these scopes, which Shelfmark needs:

| Scope | Used for |
|---|---|
| `read:catalog` | Search, and book, author and series details |
| `read:library` | Your reading status, such as Want to Read |
| `read:lists` | Your lists and the books on them |
| `read:me:content` | Test Connection and "Connected as" (also used by this app's check) |
| `read:users` | Usernames shown with lists |
| `write:library` | Setting a reading status from Shelfmark |
| `write:lists` | Removing a downloaded book from a list (on by default) |

Put the token in your **CWA profile** (it is also what CWA uses to send your
Kobo reading progress to Hardcover) and restart Calibre-Web Automated and
then this app.

## How downloads reach your library

Every download, for every Shelfmark user, is saved in the CWA ingest folder,
where CWA imports it into your library, converts it if needed and then
removes it from the folder. To keep that working, this app fixes these
Shelfmark settings:

| Setting | Value | Why |
|---|---|---|
| Output mode | Folder | Email or Grimmory upload would bypass CWA |
| Destination (books and audiobooks) | The CWA ingest folder | Also per-user destinations are overridden |
| File organization | Rename (no folders) | Shelfmark advises against folders in an ingest folder |
| Hardlink torrents | Off | CWA deletes files after import, and archives must be extracted |
| Book formats | EPUB, MOBI, AZW3, FB2, CBZ, CBR | Shelfmark's defaults without DJVU, which CWA cannot import |
| Audiobook formats | M4B, M4A, MP4 | The audiobook formats CWA imports |

Shelfmark writes each file under a temporary `.tmp` name and renames it when
it is complete; CWA skips the temporary names, so it never imports a
half-written file. Downloaded files belong to the same user as CWA's files
(uid 911), so CWA can remove them after the import.

Want different formats? Set `SUPPORTED_FORMATS` (comma-separated) in
`extra_env`, but only add formats CWA can import (for example `pdf`).

## Automatic login in Home Assistant

Opened from Home Assistant, Shelfmark logs you in automatically as the same
user as the Calibre-Web Automated app: its first admin, or the user set in
that app's `ingress_user`. A CWA admin is also a Shelfmark admin. This needs
the automatic login of the CWA app to be on (its default).

It uses Shelfmark's proxy login with a header name that is random for each
installation, stored in `.ha-ingress-login` in this app's config folder and
never written to the log. Only the Ingress proxy, which only Home Assistant
can reach, sends it; a header sent by the browser is overwritten.

Things to know:

- Every Home Assistant user who can open the panel gets this account.
- The direct port 8084 cannot be used while this is on (Shelfmark then
  expects the login header and shows no login page). To use the port, turn
  `ingress_auto_login` off; you then log in with your CWA username and
  password (`use_cwa_login`).
- After changing the CWA user (or on the very first start of CWA), restart
  this app so it picks up the change.

## Using your Hardcover Want to Read list

With a Hardcover token, Shelfmark uses Hardcover for search results and lets
you browse your own Hardcover lists, including **Want to Read**. Open the
list, pick a book and download it; Calibre-Web Automated imports it a moment
later. By default Shelfmark removes a downloaded book from that Hardcover
list. That needs a token with write access; you can turn it off in
Shelfmark's settings under Hardcover → **Auto-Remove from List on Download**.

## Automatic downloads from Want to Read

Shelfmark itself has no background automation: it only searches and
downloads when you do so in its web UI, so its **Activity** list stays empty
until something is downloaded. This app adds a small watcher that does it
for you (option `want_to_read_auto_download`, on by default). Every
`want_to_read_interval_hours` hours (default 6, first check two minutes after
start) it:

1. Fetches the **Want to Read** list from Hardcover, with the Hardcover
   token loaded from Calibre-Web Automated (your CWA profile, see "Hardcover
   token"); nothing has to be set in this app. The list is read straight
   from Hardcover; it doesn't need to be in Calibre.
2. Skips books already in your Calibre library (Shelfmark's library check).
3. Searches each other book in the sources you set up in Shelfmark (with
   your language settings) and picks a release whose title and author match,
   preferring EPUB, then the other formats CWA imports.
4. Queues it in Shelfmark. It then shows up under **Activity**, lands in the
   CWA ingest folder and is imported into your library.

At most 10 books are queued per check; the rest follow in the next check. A
book for which nothing was found is searched again after a day; a book that
was queued but is still not in the library after a week is searched again.
This is kept in `.ha-want-to-read.json` in this app's config folder (delete
it to start afresh).

## Imported books move to Currently Reading

With `want_to_read_move_when_imported` (on by default) your Hardcover lists
follow your library:

| Hardcover status | Meaning |
|---|---|
| **Want to Read** | Still to download |
| **Currently Reading** | In your Calibre library |

Every 15 minutes the app checks Want to Read; each book that is in your
Calibre library (so downloaded and successfully imported by CWA) is moved to
**Currently Reading**. A book counts as in the library when:

- Shelfmark's library check finds it (by Hardcover ID, ISBN, or title and
  author), or
- the watcher downloaded it, and a book added to Calibre since that download
  has the Hardcover title or the downloaded release title, with a matching
  author. This covers imports whose title differs from Hardcover's (a
  subtitle, a translation), which the strict library check misses.

 Hardcover keeps one
reading status per book, so it leaves Want to Read at the same moment. The
log shows `[want-to-read] 'Title': in the library, moved from Want to Read to
Currently Reading`.

- This applies to every book on Want to Read that is in your library,
  including books you added to the library yourself.
- A failed import (the file ends up in CWA's `failed` folder) leaves the book
  on Want to Read, so it is downloaded again later.
- The token needs the `write:library` scope; without it the log says so and
  the book stays on Want to Read.
- It needs the library check, which this app turns on when it finds the CWA
  library.
- This works with `want_to_read_auto_download` off too, for books you
  download by hand.

What to check when a book stays on Want to Read:

- `'Title': downloaded N hour(s) ago (...), but not found in the Calibre
  library` in the log: the download was queued, but no matching book arrived
  in Calibre. Check Shelfmark's **Activity** (did the download finish?) and
  the CWA app's config folder, `processed_books/failed` (did the import
  fail?). If the book is in Calibre under a quite different title or author,
  fix its metadata in CWA (or let CWA add a Hardcover ID, see its
  **Run Hardcover Auto-Fetch**); the next check then finds it.
- `could not move it to Currently Reading` in the log: the token misses the
  `write:library` scope.
- A book you downloaded by hand in Shelfmark is only recognised by the
  library check (Hardcover ID, ISBN, or title and author).

What to check when nothing happens:

- The log shows `Want to Read   : every 6 hour(s)` at startup and
  `[want-to-read]` lines for each check. `off (no Hardcover token)` means no
  working Hardcover token was found.
- Shelfmark needs at least one source: finish the setup wizard on first use.
- A token needs the `read:library` scope to read Want to Read.

The watcher talks to Shelfmark on localhost with an API key that is random
for this installation, stored in `.ha-api-key` in this app's config folder
and never logged. Downloads it queues are done as Shelfmark's first admin.

## Configuration

| Option | Description |
|---|---|
| `hardcover_token` | Optional. Hardcover API token. Empty (default): the token from the Calibre-Web Automated app. |
| `tor` | Route all of Shelfmark's internet traffic through Tor (default off). See "Tor" below. |
| `ingress_auto_login` | Log in automatically from Home Assistant as the Calibre-Web Automated Ingress user (default on). See above. |
| `use_cwa_login` | When not logged in automatically: log in with the Calibre-Web Automated users and their passwords (default on). Turn off to set up login in Shelfmark's settings. |
| `want_to_read_auto_download` | Download the books on your Hardcover Want to Read list automatically (default on). See above. |
| `want_to_read_interval_hours` | How often the Want to Read list is checked, in hours (1-168, default 6). |
| `want_to_read_move_when_imported` | Move books on Want to Read that are in your Calibre library to Currently Reading, every 15 minutes (default on). See above. |
| `extra_env` | Advanced: `name`/`value` pairs for other [Shelfmark environment variables](https://github.com/calibrain/shelfmark/blob/main/docs/environment-variables.md). These override the values above, including the fixed settings. |

The time zone is taken from Home Assistant.

## Tor

Turn on `tor` in the app's **Configuration** tab and restart the app to send
all of Shelfmark's internet traffic through [Tor](https://www.torproject.org/):
searches, mirrors, downloads and the Hardcover API. The sources then don't see
your IP address, and your internet provider only sees Tor traffic. Traffic
inside your own network, such as Home Assistant Ingress, is not affected.
Turn it off and restart to go back.

How it works: the image already contains Tor. At startup Shelfmark starts
Tor, waits until it is connected, and then redirects all outgoing traffic
through it with iptables, which is why the app has the `NET_ADMIN` and
`NET_RAW` permissions (Home Assistant can't grant them only when the option is
on). Shelfmark's network settings then show Tor routing as on.

Things to know:

- Starting takes up to a few minutes while Tor connects. Ingress shows a
  "starting" page in the meantime.
- If Tor does not connect, or traffic turns out not to go through Tor, the
  app stops instead of sending traffic outside Tor. Turn on **Watchdog** on
  the app's page to have Home Assistant restart it.
- Many sites block Tor or show more captchas (Cloudflare, DDoS-Guard), and
  downloads are slower. Some mirrors, or Hardcover, may not work over Tor.
- The Hardcover token check at startup runs before Tor starts.
- Tor sets the app's time zone from its exit node; this only affects times
  shown in Shelfmark's own log.

## Where the data lives

| What | Location | In Home Assistant backups |
|---|---|---|
| Shelfmark settings, users and download history | The app config folder: `app_configs/<id>_shelfmark` | Yes, with the app |
| Downloaded books | The ingest folder, until CWA has imported them | When the share/media folder is included in the backup |

## Access and security

- The app reads the config folders of your other apps **read-only**, to find
  the Calibre-Web Automated settings. It does not change them.
- The Hardcover token is never stored in this repository or written to the
  log. The Calibre-Web Automated app shares it in a file that only that app
  can read; this app reads it as root at startup.
- Port **8084** for direct access is off by default. To use it, turn on the
  port in the app's **Network** section and turn `ingress_auto_login` off,
  keep a password login enabled, and don't expose it to the internet.

## Legal note

Shelfmark can search several sources, including shadow libraries. Downloading
copyrighted books from unauthorized sources is illegal in many countries,
including the Netherlands. You are responsible for which sources you enable
and what you download.

## How it works

```
Browser / HA app ──▶ HA Ingress ──▶ nginx :8099 ──▶ Shelfmark :8084
                                                        │
                                       downloads ──▶ /share/cwa-book-ingest ──▶ CWA
```

Home Assistant serves Ingress apps under a tokenized subpath. The Shelfmark
frontend builds all its URLs from the `<base href>` in its start page; the
nginx proxy in this app puts the ingress path there.

The Calibre-Web Automated app writes its folders (`cwa.json`) and Hardcover
token to `.ha-share` in its config folder. This app maps all app config
folders read-only, finds `*_calibre_web_automated`, and turns what it finds
into Shelfmark environment variables. Shelfmark runs as the same user as
CWA (uid 911), so CWA can process and remove the downloaded files.

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

The log shows where the token came from (never the token itself). After
changing it in CWA, restart this app. Shelfmark's own Hardcover settings then
show it as set by the environment (**ENV**).

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

## Configuration

| Option | Description |
|---|---|
| `hardcover_token` | Optional. Hardcover API token. Empty (default): the token from the Calibre-Web Automated app. |
| `ingress_auto_login` | Log in automatically from Home Assistant as the Calibre-Web Automated Ingress user (default on). See above. |
| `use_cwa_login` | When not logged in automatically: log in with the Calibre-Web Automated users and their passwords (default on). Turn off to set up login in Shelfmark's settings. |
| `extra_env` | Advanced: `name`/`value` pairs for other [Shelfmark environment variables](https://github.com/calibrain/shelfmark/blob/main/docs/environment-variables.md). These override the values above, including the fixed settings. |

The time zone is taken from Home Assistant.

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

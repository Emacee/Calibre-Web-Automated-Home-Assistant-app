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

1. Make sure the **Calibre-Web Automated** app is updated to 4.0.10 or later
   and has been started once since the update. That start is when it shares
   its settings.
2. Optional: set the **Hardcover API token** in the Calibre-Web Automated
   app (get one at [hardcover.app/account/api](https://hardcover.app/account/api)),
   then restart that app.
3. Install and start this app, then click **Open web UI**. Turn on **Show in
   sidebar** to get a **Shelfmark** entry in the sidebar.
4. Log in with your **Calibre-Web Automated** username and password.
5. On first use Shelfmark shows a short setup wizard to choose where books
   are searched for.

## What is taken from Calibre-Web Automated

| What | Used for | Override in this app |
|---|---|---|
| Ingest folder | Where every download is saved, so CWA imports it | — |
| Calibre library | Marks books you already own in search results | — |
| The app itself | **Go To Library** button, opens CWA in a new tab | — |
| Hardcover API token | Hardcover search and your Hardcover lists | `hardcover_token` |
| User accounts (`app.db`) | Login to Shelfmark | `use_cwa_login: false` |

Settings set this way are locked in Shelfmark's own settings screen. Restart
this app after changing a CWA password, or after changing the Hardcover
token or folders in the Calibre-Web Automated app (restart that app first).

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
| `use_cwa_login` | Log in with the Calibre-Web Automated users (default on). Turn off to set up login in Shelfmark's settings. |
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
- Port **8084** for direct access is off by default. You can turn it on in the
  app's **Network** section, but keep the login enabled then and don't expose
  it to the internet.

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

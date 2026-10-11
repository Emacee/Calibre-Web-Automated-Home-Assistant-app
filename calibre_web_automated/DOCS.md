# Calibre-Web Automated — Documentation

This app runs [Calibre-Web Automated](https://github.com/crocodilestick/Calibre-Web-Automated)
(CWA) on your Home Assistant machine: a Calibre-Web e-book library with a web
reader, plus automatic import, conversion and metadata handling. It opens inside
Home Assistant via Ingress (optionally as **Books** in the sidebar), and can also be reached directly
on port 8083 for e-readers.

## First start

1. Install and start the app. The first start takes a minute or two.
2. Click **Open web UI** on the app's page. Turn on **Show in sidebar** to get
   a **Books** entry in the sidebar.
3. Log in with the CWA default account **`admin` / `admin123`** and change the
   password right away under **Admin → Edit users**.

On first start an empty library is created in `/share/calibre`. To use
an existing Calibre library instead, copy it (the folder containing
`metadata.db`) to that location, e.g. with the Samba share or SSH app, before
starting the app, or point `library_path` at it.

## Adding books

Drop e-books into the **ingest folder** (`/share/cwa-book-ingest` by default,
`\\homeassistant\share\cwa-book-ingest` over Samba). CWA imports them into the
library, converts them to your preferred format and fetches metadata,
depending on the settings on the **CWA Settings** page.

> **Files in the ingest folder are deleted after they are processed.** Never
> point the ingest folder at a folder whose content you want to keep. The
> app refuses to start if the ingest folder and the library overlap.

You can also upload books in the web interface.

## Configuration

| Option | Description |
|---|---|
| `library_path` | Folder of your Calibre library, under `/share` or `/media`. Default `/share/calibre`. |
| `ingest_path` | Ingest folder, under `/share` or `/media`. Default `/share/cwa-book-ingest`. Its content is **deleted** after import. |
| `hardcover_token` | Optional [Hardcover API token](https://docs.hardcover.app/api/getting-started/) to use Hardcover as a metadata provider. |
| `network_share_mode` | Turn on when the library or ingest folder is on a network share (an NFS/SMB storage mounted under **Settings → System → Storage**, which shows up in `/media` or `/share`). Disables SQLite WAL and ownership changes, and uses polling for the ingest folder. |
| `watch_mode` | `inotify` (default) reacts to new files immediately. `poll` scans periodically; use it when new files are not picked up, e.g. on network shares. |
| `disable_library_automount` | Skip the startup scan that looks for `metadata.db` in the library folder and configures it automatically. |
| `extra_env` | Advanced: `name`/`value` pairs for other CWA environment variables, e.g. `TRUSTED_PROXY_COUNT`. |

Restart the app after changing options. The time zone is taken from Home
Assistant.

## Where the data lives

| What | Location | In Home Assistant backups |
|---|---|---|
| CWA settings, users, `app.db`, `cwa.db`, processed-book archive and logs | The app config folder: `app_configs/<id>_calibre_web_automated` (`addon_configs` in older versions of the Samba and SSH apps) | Yes, with the app |
| Library | `library_path` | When the share/media folder is included in the backup |
| Ingest folder | `ingest_path` | When the share/media folder is included in the backup |

Calibre plugins can be placed in `.config/calibre/plugins` inside the app
config folder.

## Direct access, OPDS and Kobo sync

Port **8083** is open by default at `http://homeassistant.local:8083`. E-readers
cannot use Ingress (it needs a Home Assistant login), so you need this port for:

- OPDS catalogue: `http://homeassistant.local:8083/opds`
- Kobo sync (enable it in **Admin → Edit Basic Configuration → Feature
  Configuration**, then follow the CWA wiki)
- KOReader sync

If you only use the browser, you can disable the port in the app's
**Network** section. Don't expose port 8083 to the internet without a reverse
proxy with HTTPS.

## Logging in with your Home Assistant user (optional)

Home Assistant Ingress sends the logged-in user in the `X-Remote-User-Name`
header. Calibre-Web can use this to log you in automatically: under **Admin →
Edit Basic Configuration → Feature Configuration** enable **Allow Reverse
Proxy Authentication** with header name `X-Remote-User-Name`, and create CWA
users whose usernames match your Home Assistant usernames.

**Only do this when port 8083 is disabled.** On the direct port, anyone on your
network could send that header themselves and log in as any user.

## How it works

Home Assistant serves Ingress apps under a tokenized subpath. A small nginx
proxy inside the app passes that path to Calibre-Web as `X-Script-Name`,
which Calibre-Web supports natively. A few CWA-specific pages (statistics,
duplicates) build URLs in JavaScript; nginx rewrites those to include the
prefix.

```
Browser / HA app ──▶ HA Ingress ──▶ nginx :8099 ──▶ CWA :8083
E-reader (OPDS/Kobo) ─────────────────────────────▶ CWA :8083
```

The upstream image declares Docker volumes for `/config`, `/calibre-library` and
`/cwa-book-ingest`. Under the Supervisor those would become anonymous volumes that
are lost when the app container is recreated, so the app's Dockerfile
copies the upstream image into a new image without them. `/calibre-library` is a
link to `library_path`, and CWA's ingest folder is set to `ingest_path`.

## Known limitations

- Building the app image on the device takes several minutes and needs
  several GB of free disk space (the CWA image includes Calibre).
- If a CWA update renames frontend routes, a page may not work through Ingress
  until the app is updated; the direct port keeps working.

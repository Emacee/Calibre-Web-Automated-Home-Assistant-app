# Calibre-Web Automated — Documentation

This app runs [Calibre-Web Automated](https://github.com/crocodilestick/Calibre-Web-Automated)
(CWA) on your Home Assistant machine: a Calibre-Web e-book library with a web
reader, plus automatic import, conversion and metadata handling. It opens inside
Home Assistant via Ingress (optionally as **Calibre** in the sidebar), and can also be reached directly
on port 8084 for e-readers.

## First start

1. Install and start the app. The first start takes a minute or two.
2. Click **Open web UI** on the app's page. Turn on **Show in sidebar** to get
   a **Calibre** entry in the sidebar.
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
| `ingress_auto_login` | Log in automatically when opened from Home Assistant (default on). See below. |
| `ingress_user` | Optional. CWA user for the automatic login. Empty: the first admin user. |
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

## Downloading books with Shelfmark

The **Shelfmark** app from this repository searches and downloads books, also
from your Hardcover lists such as Want to Read, into the ingest folder of
this app. On every start this app writes its library and ingest folder, and
the `hardcover_token` if set, to `.ha-share` in its config folder (the token
file is only readable by the app itself). It also shares the user of the
automatic login below (name and admin role, not the secret header), so
Shelfmark logs you in as the same user. Shelfmark reads them from there, so it
needs no setup of its own. Restart Shelfmark after changing these options here.

## Direct access, OPDS and Kobo sync

Port **8084** is open by default at `http://homeassistant.local:8084`. E-readers
cannot use Ingress (it needs a Home Assistant login), so you need this port for:

- OPDS catalogue: `http://homeassistant.local:8084/opds`
- Kobo sync (enable it in **Admin → Edit Basic Configuration → Feature
  Configuration**, then follow the CWA wiki)
- KOReader sync

If you only use the browser, you can disable the port in the app's
**Network** section. Don't expose port 8084 to the internet without a reverse
proxy with HTTPS.

## Automatic login in Home Assistant

When you open the app from Home Assistant (Ingress), you are logged in
automatically as the CWA admin, without a login page. The first admin user is
used, or the user set in `ingress_user`. E-readers and browsers on the direct
port 8084 still have to log in with a password.

How it works: Calibre-Web's reverse proxy login is switched on with a header
name that is random for each installation, stored in `.ha-ingress-login` in
the app config folder. Only the Ingress proxy, which only Home Assistant can
reach, sends that header, so it cannot be forged on port 8084. The header
name is never written to the log. On the very first start Calibre-Web
restarts once to apply the setting.

Things to know:

- Every Home Assistant user who can open the panel gets this CWA account.
  With several Home Assistant users, set `ingress_user` to a non-admin CWA
  user, or turn `ingress_auto_login` off.
- Logging out has no effect in Home Assistant while this is on.
- It replaces any reverse proxy login you configured yourself in CWA. Turning
  `ingress_auto_login` off switches only this app's own setup off.

## How it works

Home Assistant serves Ingress apps under a tokenized subpath. A small nginx
proxy inside the app passes that path to Calibre-Web as `X-Script-Name`,
which Calibre-Web supports natively. A few CWA-specific pages (statistics,
duplicates) build URLs in JavaScript; nginx rewrites those to include the
prefix.

```
Browser / HA app ──▶ HA Ingress ──▶ nginx :8099 ──▶ CWA :8084
E-reader (OPDS/Kobo) ─────────────────────────────▶ CWA :8084
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

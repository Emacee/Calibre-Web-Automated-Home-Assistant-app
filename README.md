# Calibre-Web Automated for Home Assistant

Home Assistant app (add-on) for [Calibre-Web Automated](https://github.com/crocodilestick/Calibre-Web-Automated)
(CWA): an e-book library with a web reader, automatic import, conversion and
metadata handling of new books, OPDS and Kobo sync. It opens in the Home
Assistant sidebar through Ingress, and stays reachable on port 8083 for
e-readers.

## Installation

[![Add repository to my Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FEmacee%2FCalibre-Web-Automated-Home-Assistant-app)

Or manually:

1. In Home Assistant go to **Settings → Apps → App store**.
2. Open the **⋮** menu → **Repositories** and add
   `https://github.com/Emacee/Calibre-Web-Automated-Home-Assistant-app`.
3. Install **Calibre-Web Automated** and start it. The image is built on your
   device, which takes several minutes the first time.
4. Open **Books** in the sidebar and log in with `admin` / `admin123`, then
   change that password right away.

Supported architectures: `amd64` and `aarch64`.

See the [app documentation](calibre_web_automated/DOCS.md) for the
configuration options, where data is stored, OPDS/Kobo access and how the
Ingress support works.

## Apps

| App | Version | Based on |
|---|---|---|
| [Calibre-Web Automated](calibre_web_automated) | 4.0.8.1 | [crocodilestick/calibre-web-automated](https://hub.docker.com/r/crocodilestick/calibre-web-automated) v4.0.8 |

## Credits

This is an unofficial app and not affiliated with the Calibre-Web Automated
project. All credit for the application goes to
[CrocodileStick and the CWA contributors](https://github.com/crocodilestick/Calibre-Web-Automated),
which builds on [Calibre-Web](https://github.com/janeczku/calibre-web) and
[Calibre](https://calibre-ebook.com/). Report problems with the app itself
(installation, Ingress, options) here, and problems with CWA upstream.

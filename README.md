# Pellu Home Server for Linux and NAS

Pellu Home Server shares the videos in the folders you choose with the Pellu apps on your phones,
tablets and TV at home. Files play as they are: nothing is converted, and nothing is uploaded to
Pellu. You set it up from a settings page in your browser.

This repository holds this README and an example `docker-compose.yml` for the container image
`ghcr.io/yonikrispil/pellu-home`. The full guide, with steps for each NAS, is at
https://pellu.co/home-server/linux.

## What's in the image

- `pellu-home`, the server: one native binary (no runtime to install), serving the Pellu apps'
  API, the videos themselves (with seeking) and their subtitles on port 8484.
- Its settings page, at `http://<address>:8484/admin`: folders, linking to your Pellu account,
  the network and a password.
- Bonjour (`_pellu-home._tcp`), so the apps find the server on the home network by themselves.
- A health check (`pellu-home --health`) and an entrypoint that runs the server as `PUID:PGID`,
  never as root.
- Built on `debian:trixie-slim`, for `linux/amd64` and `linux/arm64` (a Raspberry Pi 4 or 5 with
  a 64-bit OS works).

## Quick start

You need a NAS or a Linux computer with Docker that stays on while you watch, on the same home
network as your devices, and a free Pellu account.

1. Save [`docker-compose.yml`](docker-compose.yml) in a folder of its own. Change `/path/to/media`
   to your media folder, and `PUID` and `PGID` to the user and group that own it (`id` shows them).
2. Start it there:

   ```sh
   docker compose up -d
   ```

3. Find the setup code in the log:

   ```sh
   docker logs pellu-home
   ```

   It's a line like `Pellu Home Server: open http://192.168.1.50:8484/admin and enter the setup code ABCD-EFGH`.
4. Open that address on a computer or phone at home, enter the code and choose a password.
5. Add your media folders (with Docker, they are the ones under `/media`), then choose
   **Link to account** and type the code it shows in your account at https://pellu.co.
6. Open Pellu on a phone, tablet or TV at home: the server's folders show up by themselves.

More folders? Add a line for each under `volumes`, inside `/media`, for example
`/volume1/shows:/media/shows:ro`.

### Without host networking

Some systems don't allow `network_mode: host`. Then publish the port and give the server its
address on your home network, or your devices won't reach it:

```yaml
    ports:
      - "8484:8484"
    environment:
      - PELLU_ADVERTISE_ADDRESS=192.168.1.50:8484
```

Bonjour doesn't work this way, so the apps find the server through your account.

To use the NAS's own Avahi for Bonjour instead of the server's built-in responder, also mount
`/run/dbus/system_bus_socket:/run/dbus/system_bus_socket`.

## Settings

| Path or variable | What it is |
|---|---|
| `/config` | The settings (`settings.json`). Keep it on a volume. |
| `/media` | Your media folders, mounted read-only. Folders are picked in the settings page from under here. |
| `8484/tcp` | The settings page and the apps. |
| `PUID`, `PGID` | The user and group the server runs as (1000 unless set). |
| `PELLU_NAME` | The name the Pellu apps show. |
| `PELLU_ADVERTISE_ADDRESS` | Addresses sent to your account (`ip` or `ip:port`, comma-separated), for a bridge network. |
| `PELLU_PORT` | Another port instead of 8484. |
| `PELLU_SCAN_MINUTES` | A full scan every so many minutes (at least 15; 0: only at start and on demand). |
| `PELLU_LANG` | `en` or `he`: the log's language and the settings page's first language. |
| `PELLU_MDNS` | Bonjour: `auto`, `avahi`, `builtin` or `off`. |
| `PELLU_LOG_LEVEL` | `debug`, `info`, `warn` or `error`. |
| `TZ` | Your time zone. |

What's set in the environment is shown in the settings page but can't be changed there. Everything
else is set in the page.

## Updating

```sh
docker compose pull && docker compose up -d
```

The settings in `/config` are kept.

## Help

Questions or problems: choose Support in your account at https://pellu.co.

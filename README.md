# sonos-cli

Minimal command-line control for a single Sonos speaker, addressed by IP.

Written because Sonos has no Linux client and the official web app
(`play.sonos.com`) covers playback but not EQ. This talks to the speaker's local
UPnP API instead, so volume, bass, treble and loudness are all reachable from a
terminal.

## Requirements

- [`uv`](https://docs.astral.sh/uv/) — the script is a self-contained inline
  script; `uv` resolves and caches the `soco` dependency itself, so there is no
  venv to manage.
- Network access to the speaker on port 1400 (plain HTTP, LAN only).

## Install

```sh
git clone <this repo> ~/Projects/sonos-cli   # or use the existing checkout
ln -sfn ~/Projects/sonos-cli/sonos ~/.local/bin/sonos
```

`~/.local/bin` is expected to be on `PATH`.

## Usage

```
sonos status                     volume, mute, bass, treble, loudness, transport state
sonos vol                        read volume
sonos vol 25                     absolute volume (0-100)
sonos vol +5 / sonos vol -5      relative
sonos mute on|off|toggle
sonos bass [N|+N|-N]             -10..10
sonos treble [N|+N|-N]           -10..10
sonos loudness on|off|toggle
sonos play|pause|stop|next|previous|toggle
sonos ip                         show the configured speaker IP
sonos ip 192.168.1.50            point the CLI at another speaker
sonos --help
```

Writes are verified with a read-back retry loop, so a reported value is what the
speaker actually holds.

## Configuration

The speaker IP lives in `~/.config/sonos-cli/config.json`:

```json
{
  "ip": "192.168.1.113"
}
```

Override the path with `SONOS_CLI_CONFIG=/some/other.json sonos status`.
`sonos ip <address>` writes this file for you.

## Notes / caveats

- **Discovery is deliberately never used.** `soco.discover()` relies on SSDP
  multicast, which is filtered on this network (the AP drops multicast between
  clients), so the IP is pinned in the config file instead.
- **"Volume limit" is not possible here.** The speaker's advertised UPnP
  services (`RenderingControl1`, `DeviceProperties1`, `GroupRenderingControl1`,
  `AVTransport1`, `MusicServices1`) contain no `limit` entries — only
  `Bass`/`Treble`/`Loudness`/`EQ`/`NightMode`/`Volume`. Volume limit, along with
  Trueplay tuning and firmware updates, is phone-app only.
- **soco 0.32 specifics** baked into the script:
  - `SoCo()` accepts no `timeout` argument; the timeout is set globally via
    `soco.config.REQUEST_TIMEOUT`.
  - `soco.config.CACHE_ENABLED` defaults to `True`, so a read immediately after
    a write returns the stale value — the script disables caching and confirms
    each write with a read-back retry loop.
- **Value ranges:** volume 0-100, bass -10..10, treble -10..10. Out-of-range
  input is clamped, not rejected.
- One speaker per invocation; grouping and queue control are out of scope.

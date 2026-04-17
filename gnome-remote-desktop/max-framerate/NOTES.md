# gnome-remote-desktop: configurable max-framerate

## Summary

Adds a `max-framerate` GSettings key under
`org.gnome.desktop.remote-desktop.rdp`, wires it through `GrdSettings` and
its subclasses, and uses it to cap the PipeWire screencast stream's
`maxFramerate` property. Previously hardcoded as `TARGET_SURFACE_REFRESH_RATE`
at 60 fps.

## Motivation

On virtio-gpu VMs without H.264 encode acceleration, GRD falls back to
software RFX Progressive encoding (CPU-only RLGR1 wavelet codec). At 1440p
× 60 fps, this saturates a CPU core and produces visible stutter. Capping
PipeWire's negotiated rate halves the encoding load and makes remote
sessions usable without touching the encoder itself.

Related upstream bug: [Ubuntu #2103988](https://bugs.launchpad.net/bugs/2103988)
(GRD software encoding performance, landing in Ubuntu 26.04).

## Links

- **Upstream MR:** https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/merge_requests/396
- **Fork:** https://gitlab.gnome.org/catinspace-au/gnome-remote-desktop
- **Branch:** `feat/configurable-max-framerate`
- **Base:** `origin/main` at time of format-patch (see commit SHA in each patch)
- **Build tree (Ubuntu 49.0 deb):** `/projects/gnome-remote-desktop-build/`
- **Built deb:** `gnome-remote-desktop_49.0-0ubuntu1.1hyperi1_amd64.deb`

## Commits

1. `0001` — Add `max-framerate` key to RDP gsettings schemas
2. `0002` — Add `rdp-max-framerate` GObject property to `GrdSettings`
3. `0003` — Wire max-framerate through all settings subclasses
4. `0004` — Read max-framerate from settings instead of hardcoded define

## Verified behavior (desktop-derek, 2026-03-25)

```
gsettings set org.gnome.desktop.remote-desktop.rdp max-framerate 30
```

PipeWire stream negotiates `maxFramerate: 30/1` on both gnome-shell output
(node 58) and GRD input (node 63). Confirmed via `pw-cli enum-params <id> Format`.

## Status

| | |
|---|---|
| MR opened | 2026-03-25 |
| Human comments | 0 |
| CI status | Jobs skipped — GNOME GitLab requires maintainer approval for first-time contributor pipelines |
| Next action | Ping `#remote-desktop:gnome.org` Matrix room if no movement by ~2026-05-01 |

## Regenerating patches

From `/projects/gnome-remote-desktop/` (submodule `sources/gnome-remote-desktop`):

```bash
git format-patch origin/main..feat/configurable-max-framerate \
    --output-directory /path/to/hyperi-linux-upstream-patches/gnome-remote-desktop/max-framerate/
```

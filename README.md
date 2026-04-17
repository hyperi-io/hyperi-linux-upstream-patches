# hyperi-linux-upstream-patches

Tracking repo for upstream contributions across the Linux desktop/server stack
originating from hyperi.io infrastructure work. Patches live here as
`git format-patch` output; source trees are referenced as submodules under
`sources/`.

This is **not** a fork of any upstream project. It's an index + archive.

## Status

| Area | Contribution | Upstream | MR/Patch | Status |
|---|---|---|---|---|
| `gnome-remote-desktop` | [`max-framerate`](gnome-remote-desktop/max-framerate/) | GNOME / GitLab | [MR !396](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/merge_requests/396) | Open, awaiting first-time contributor CI approval (22 days) |
| `linux` (kernel) | [`virtio-gpu-wq-unbound`](linux/virtio-gpu-wq-unbound/) | `dri-devel@lists.freedesktop.org` | _not yet submitted_ | Design stage |

## Repository layout

```
.
├── gnome-remote-desktop/        # GNOME project contributions
│   └── max-framerate/
│       ├── 0001-*.patch         # git format-patch output
│       └── NOTES.md             # context, links, build notes
├── linux/                       # Linux kernel contributions
│   └── virtio-gpu-wq-unbound/
│       └── NOTES.md
└── sources/                     # Submodules pointing at upstream forks
    ├── gnome-remote-desktop     → gitlab.gnome.org/catinspace-au/gnome-remote-desktop
    └── linux                    → github.com/hyperi-io/linux
```

## Working with this repo

```bash
# Clone with submodules (populates sources/)
git clone --recursive https://github.com/hyperi-io/hyperi-linux-upstream-patches
# or after a shallow clone
git submodule update --init --recursive
```

Submodules pin to specific SHAs for reproducibility. When you rebase a
feature branch in a source fork, bump the submodule pointer here and
regenerate the corresponding `.patch` files.

## Why patches-only (not source mirror)

- GNOME requires MR submission from a GitLab fork — can't migrate that to GitHub.
- Linux kernel uses `git send-email` to mailing lists — GitHub PRs are not the submission path for mainline/DRM.
- A mirror of either upstream tree here would be dead weight and confusing.

Submodules + patch archive lets us track *what was contributed and where*
without pretending this repo is authoritative for any codebase.

## Note on namespaces

The GitHub org is `hyperi-io` (this repo, plus `hyperi-io/linux`).

The GNOME fork lives at `gitlab.gnome.org/catinspace-au/gnome-remote-desktop`
rather than under `hyperi-io`, because gitlab.gnome.org admin-gates top-level
group creation (non-admins get `can_create_group: False`). The user namespace
`catinspace-au` is where MR !396 was authored and remains the stable path.
Cross-platform org alignment would cost the 22-day review timestamp on !396
for no functional gain.

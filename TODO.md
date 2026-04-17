# TODO

Rolling list of next actions for this tracking repo and its contributions.

## virtio_gpu WQ_UNBOUND patch (Linux kernel)

Write the patch, get it reviewed, submit upstream. Full design and
submission checklist lives in [`linux/virtio-gpu-wq-unbound/NOTES.md`](linux/virtio-gpu-wq-unbound/NOTES.md).
High-level sequence:

- [ ] Search `lore.kernel.org` and `git log drivers/gpu/drm/virtio/` in `sources/linux` for prior WQ_UNBOUND submissions — skip the work if someone already sent it
- [ ] Identify every bound `alloc_workqueue()` call site in `drivers/gpu/drm/virtio/`
- [ ] Write the patch against `drm-misc-next` (branch in `sources/linux`)
- [ ] Build a test kernel or DKMS module; validate on `desktop-derek` with before/after dmesg + latency evidence
- [ ] Run `scripts/checkpatch.pl --strict` until clean
- [ ] Run `scripts/get_maintainer.pl drivers/gpu/drm/virtio/` for the CC list
- [ ] Configure `git send-email` (SMTP auth, one-time setup)
- [ ] Send to `dri-devel@lists.freedesktop.org`, CC Gerd Hoffmann and `virtualization@lists.linux.dev`
- [ ] `git format-patch` the final version into `linux/virtio-gpu-wq-unbound/` and commit
- [ ] Update status table in `README.md` and `linux/virtio-gpu-wq-unbound/NOTES.md` with the lore.kernel.org message URL

## GRD max-framerate MR !396

- [ ] If no movement by ~2026-05-01, post a polite ping in `#remote-desktop:gnome.org` Matrix
- [ ] If CI approval lands, rebase on upstream `main` if needed and regenerate patches in `gnome-remote-desktop/max-framerate/`

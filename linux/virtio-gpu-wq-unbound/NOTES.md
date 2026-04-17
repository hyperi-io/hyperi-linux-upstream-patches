# linux/drm/virtio: use WQ_UNBOUND for resource cleanup workqueues

**Status:** design / not yet patched / not yet submitted

## Problem

On virtio-gpu guests under heavy guest-CPU load (e.g. software video
encoding in GNOME Remote Desktop), the virtio_gpu kernel driver's
per-CPU (bound) workqueues stall, blocking frame delivery and host-guest
virtio command processing.

Guest dmesg, reliably reproducible on desktop-derek:

```
workqueue: virtio_gpu_array_put_free_work [virtio_gpu] hogged CPU for >10000us 4 times, consider switching to WQ_UNBOUND
workqueue: virtio_gpu_array_put_free_work [virtio_gpu] hogged CPU for >10000us 5 times
workqueue: virtio_gpu_array_put_free_work [virtio_gpu] hogged CPU for >10000us 7 times
workqueue: virtio_gpu_array_put_free_work [virtio_gpu] hogged CPU for >10000us 11 times
```

Counts escalate on the kernel's doubling threshold (4 → 5 → 7 → 11 → 22 → ...)
over uptime as more cleanup work queues up behind a busy CPU.

## Root cause

`virtio_gpu_array_put_free_work` and `virtio_gpu_dequeue_ctrl_func` are
queued onto bound (per-CPU) workqueues. When the CPU that received the
release callback is saturated by an unrelated workload (in our case, GRD's
software RFX encoder), the work item waits in that CPU's runqueue for
10–100ms even though 30+ other vCPUs are idle. The bound workqueue cannot
migrate the work. During the wait:

- Freed resource arrays aren't returned to the pool → subsequent frame
  allocations may block
- The host→guest response ring isn't drained → QEMU's virtqueue backs up
  → host stops accepting new commands from guest
- Visible as session-wide stutter, delayed cursor, keyboard lag

## Proposed fix

Change the relevant `alloc_workqueue()` calls in `drivers/gpu/drm/virtio/`
to pass `WQ_UNBOUND`, allowing the scheduler to dispatch work items to any
available CPU. The kernel's own workqueue infrastructure prints the
`consider switching to WQ_UNBOUND` suggestion as targeted advice for
exactly this pattern.

## TODOs before submission

- [ ] Locate all bound `alloc_workqueue()` call sites in `drivers/gpu/drm/virtio/`
      (at least `virtgpu_object.c`, possibly `virtgpu_kms.c`)
- [ ] Search lore.kernel.org and `git log drivers/gpu/drm/virtio/` for prior
      WQ_UNBOUND submissions — do not duplicate existing work
- [ ] Write the patch against `drm-misc-next` (see `sources/linux` for working tree)
- [ ] Run `scripts/checkpatch.pl --strict` on the patch
- [ ] Build and install a test kernel (DKMS module is enough to prove the patch)
- [ ] Collect before/after dmesg and perceived-latency evidence
- [ ] Run `scripts/get_maintainer.pl drivers/gpu/drm/virtio/virtgpu_object.c`
      to build the CC list
- [ ] Send via `git send-email` to `dri-devel@lists.freedesktop.org`,
      CCing Gerd Hoffmann and `virtualization@lists.linux.dev`

## Links

- **Fork (eventual patch home):** https://github.com/hyperi-io/linux (GitHub mirror — not the submission path)
- **Primary submission:** mailing list — `dri-devel@lists.freedesktop.org`
- **Related downstream doc:** `/projects/hyperi-infra/docs/RDP-FIX.md`

## Rollout for Derek's fleet (pre-upstream)

Once the patch is validated locally, package it as a DKMS module so it
rebuilds automatically on kernel upgrades and ships via the `rdp` Ansible
role. Track in `/projects/dfe-developer/ansible/roles/rdp/`. Drop the DKMS
package once the patch lands in mainline and filters through to Ubuntu
stable.

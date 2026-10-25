# postmarketOS on Samsung Galaxy J3 (2017), SM-J330F (WIP, downstream)

Work-in-progress postmarketOS port for the Samsung Galaxy J3 (2017), SM-J330F, Exynos 7570.
Downstream Samsung kernel 3.18.140. Tested on one device only.

The codename `j3y17lte` is inferred from sibling devices (`j5y17lte`, `j7y17lte`) and
from `j3y17` in the Samsung kernel sources. It is not confirmed from the phone itself.

## Project state: downstream, looking for help with mainline

This port runs on Samsung's old downstream 3.18 kernel. That is a dead end long-term:
the kernel is unmaintained and the patches are specific to it. The owner is not planning
to do the mainline work alone, because it is long and hard, and downstream work on this
phone is no longer interesting to him.

The goal is for this to be picked up by other people and moved towards mainline Linux.
I have not checked how much Exynos 7570 support mainline has today, so verify that first.
What this repo gives you is a working starting point and notes about what the hardware
actually needs (panel timings, backlight power, touchscreen chip and address, pitfalls).

If you have this phone or a similar Exynos 7570 device and want to continue:
fork this repo, open an issue here, or reuse the findings in your own project.

## Status

Works (observed on the device):
- Display (TD4100 panel) and backlight
- Touchscreen (Synaptics TD4100), correct axes, no DTS flip needed
- Graphical session: X11 on `fbdev` (no `/dev/dri`, no GPU acceleration), openbox and XFCE ran
- USB networking, ssh into the phone

Not working / not tried:
- Wi-Fi (firmware for `mx140` not found), Bluetooth (not tested)
- Audio, modem, cameras, sensors, NFC, vibration
- Touch IC firmware update is intentionally skipped (patch 0018)

## What is in this repo

- `patches/0009..0019`: my patches on top of the pmaports `linux-samsung-j4lte` package
- `package/APKBUILD`, `package/config-samsung-j4lte.aarch64`: the modified package files

Patches 0001-0008 and `fix-check-lxdialog.patch` come from pmaports and are not included.
This port is built on the `j4lte` package by its maintainer Hendra Manudinata. Thanks.

## Building

The device is currently built as `samsung-j4lte` in pmbootstrap (a separate
`device-samsung-j3y17lte` package does not exist yet).

1. Copy `patches/*.patch`, `package/APKBUILD` and `package/config-samsung-j4lte.aarch64`
   into `device/downstream/linux-samsung-j4lte/` of your pmaports checkout.
2. `pmbootstrap checksum linux-samsung-j4lte` (the tarball hash in APKBUILD may differ).
3. `pmbootstrap build linux-samsung-j4lte`, then install and export the boot image.

Flashing can brick your phone. You do it at your own risk.

## Known pitfalls

- Do not enable `CONFIG_SEC_INCELL` without checking `<linux/sec_incell.h>`; patch 0019 exists to avoid it.
- `bwrap` does not work on this kernel. gdk-pixbuf 2.44 loads images through glycin,
  which needs bwrap, so xfce4-panel and xfdesktop crash on icons. A wrapper script named
  `bwrap` that drops the options and runs the program directly worked around it.
- `/dev/fb0` only works through `mmap`; `read`/`write` (`cat`, `dd`) hang or return nothing.
- The screen can go black when X starts; `echo 0 > /sys/class/graphics/fb0/blank` brought it back.
- After a reboot, XFCE did not start by itself in the last session. The cause was not found
  (a guess: the `bwrap` wrapper is not on the PATH of the boot-time X process).

## Authorship and testing

Most of the code in this repo (the patches and the package changes) was written by an AI
assistant (Claude, by Anthropic). All testing was done by the repo owner on a real
SM-J330F: flashing, running, observing the results and reporting them back. The status
list above comes from those observations.

The patches have not been reviewed by kernel or postmarketOS maintainers. AI-written
kernel code can be wrong in ways that are not obvious, so read the patches before use.

## License

Patches are licensed under GPL-2.0-only, like the Linux kernel.

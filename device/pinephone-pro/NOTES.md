<!-- SPDX-License-Identifier: MIT -->
# PinePhone Pro: external community integration

This documents an experimental integration on the PinePhone Pro (Rockchip
RK3399S), including the official PinePhone Keyboard. It is not a port for the
original PinePhone, and does not add an installer target to this repository.

The integration code is available under MIT in
[omarchy-mobile-pinephone-pro at 0afd6bf3e515573f79b699d0a80ed7bbdd678fbf](https://github.com/FlorencioJP/omarchy-mobile-pinephone-pro/tree/0afd6bf3e515573f79b699d0a80ed7bbdd678fbf).
The intended use is a portable SSH client, with agents and persistent tmux
sessions on remote machines. Credentials are configured locally.

## Hardware base and scope

The integration keeps the PinePhone Pro kernel, device tree, drivers and boot
chain from [DanctNIX / Pine64-Arch](https://github.com/dreemurrs-embedded/Pine64-Arch).
Omarchy's user interface is layered onto that base; it does not use the Pixel
kernel, Android partition layout, vendor provisioning or fastboot procedures.
No SPI/Tow-Boot change was made during this integration.

There is deliberately no `device.conf` here: `tools/install-phone` requires
slot-B boot images and `userdata` in its storage allowlist. Those are not the
PinePhone Pro's installation model. Do not use it, `bin/omamobile flash`, or
the Pixel root images on this device.

The public integration contains package locks, configuration helpers, input
mode selection and tests. It is **not a complete OS installer**: its initial
experimental base image and a complete builder for that image are not publicly
available. A fresh installation from the public checkout alone has not been
replayed on hardware. This note is a port report, not a flash runbook.

## 2026-10-24: external integration results

The following summarizes the public
[status report](https://github.com/FlorencioJP/omarchy-mobile-pinephone-pro/blob/0afd6bf3e515573f79b699d0a80ed7bbdd678fbf/CURRENT-STATUS.md)
and [README](https://github.com/FlorencioJP/omarchy-mobile-pinephone-pro/blob/0afd6bf3e515573f79b699d0a80ed7bbdd678fbf/README.md).
These are results from one owner's device, recorded before this documentation
contribution; they have not been independently repeated for this PR. Hardware
observations below are separate from successful parsing, package installation
or host-side tests.

| Component | Recorded version |
| --- | --- |
| Kernel | `6.18.55-1-megi-danctnix` |
| Omarchy | `4.0.6-1` |
| Hyprland | `0.56.2-7` |
| Quickshell | `0.3.2.r7.g2b4501a-1` |
| Omarchy source reviewed for the integration | `f643adc38ad65c4900609c5c1b0b5c359fef147a` |

The [version lock](https://github.com/FlorencioJP/omarchy-mobile-pinephone-pro/blob/0afd6bf3e515573f79b699d0a80ed7bbdd678fbf/upstream.lock.json)
does not pin a complete Omarchy Mobile base-image build. These versions are
the tested snapshot, not a compatibility claim for current upstream packages.

| Area | Recorded evidence | Remaining limits |
| --- | --- | --- |
| microSD boot | Owner confirmed boot and login. | No public image or end-to-end image rebuild. |
| eMMC boot | Later file-based migration preserved the partition table and bootloader; desktop, Wi-Fi and maintenance SSH worked from eMMC with the SD unmounted. | SD was still physically inserted; boot with it removed was not tested. This is not an instruction to migrate another device. |
| Display and GPU | Hyprland desktop displayed; renderer reported `Mali-T860 MC4` / Panfrost. | No benchmark or endurance test. |
| Touch without keyboard | Portrait SDDM login and Squeekboard typing confirmed on SD. | A fully touch-driven first boot from a clean image remains incomplete. |
| Official keyboard | Physical typing and automatic landscape mode observed after boot with the keyboard attached. | Exhaustive modifiers, arrows and symbol combinations remain untested. Hotplug is not guaranteed. |
| Desktop | Mobile bar, Omarchy menu and theme selection confirmed; automatic terminal launch removed. | Full desktop Omarchy feature parity is not claimed. The touch quick-settings menu tile was deployed locally and has no public installer yet. |
| Wi-Fi and SSH | Wi-Fi association and maintenance SSH used on the device. | Resume/reconnection and access to the owner's remote agent hosts not validated. |
| Battery reporting | Read-only table for phone and keyboard batteries confirmed; both later reported Full/100%. | This does not validate charging: phone discharge had previously been observed while charging through the keyboard. No controlled charge/discharge cycle. |
| Bluetooth and audio | Services and settings tools available. | Pairing, playback and capture untested. |
| Cellular data | Modem detected; ModemManager/eg25-manager and connection editor present. | No SIM inserted; registration and data untested. Calls and SMS outside the initial scope. |
| Suspend and autonomy | Kernel exposes deep sleep; shell idle timeouts disabled in the recorded setup. | No suspend/resume cycle or battery-life validation. |

## Input and power lessons

The external [input-mode implementation](https://github.com/FlorencioJP/omarchy-mobile-pinephone-pro/blob/0afd6bf3e515573f79b699d0a80ed7bbdd678fbf/INPUT-MODES.md)
selects portrait (`transform = 0`) without the keyboard and landscape
(`transform = 3`) with it, applying the same transform to the touchscreen.
Detection occurs at session start; a stale sysfs input entry after keyboard
removal means it must not be advertised as reliable hotplug detection.

Further keyboard validation must cover Ctrl, Alt, Esc, Tab, arrows, pipe,
tilde, slash, backslash, braces and brackets in a terminal and remote tmux.
Host-side keymap tests are not evidence that those physical combinations work.

Follow the [PINE64 keyboard power instructions](https://pine64.org/documentation/Phone_Accessories/Keyboard/):
with the keyboard mounted, charge through the keyboard's USB-C port and do not
connect a charger to the phone at the same time. Battery status alone does not
prove that power transfer or the keyboard button behaves correctly.

## Work needed before an upstream installer target

1. Publish a reproducible microSD base-image build, with verified dependencies,
   retaining the DanctNIX boot files and hardware support. Start on microSD;
   keep internal storage and the existing bootloader untouched.
2. Package the PinePhone-specific session and input configuration separately
   from Pixel hardware services, then test a clean installation both with and
   without the official keyboard. Stop if the boot target is ambiguous; recover
   a microSD-only trial by powering down and removing the card.
3. Record the complete keyboard matrix, charging behavior, suspend/resume,
   Wi-Fi reconnection and autonomy on hardware before widening support claims.

No firmware blobs, device captures, private recovery images or local Hyprperks
adaptations are included in this contribution. Dependency licenses remain
separate from the MIT license of the integration helpers.

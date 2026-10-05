# ansible-role-hardware

Makes a recent laptop/PC of any make work as far as the distribution supports it: installs all firmware and
CPU microcode the enabled repositories offer, the hardware drivers and userspace hardware support, and the VM
guest agents. Supported: openSUSE Leap/Tumbleweed, Debian, Fedora, EL (AlmaLinux, Rocky).

Run after ansible-role-repo with `repo.distribution_nonfree: true` (Leap non-oss, Debian
contrib/non-free/non-free-firmware, EL CRB + EPEL); with `repo.rpmfusion: true` firmware and drivers from
RPM Fusion are picked up as well.

## What is installed

- **firmware**: the distribution's meta package (Leap: pattern `hardware` = `kernel-firmware-all` + AMD/Intel
  microcode; Debian: `firmware-linux`; Fedora: `linux-firmware` + group `hardware-support`; EL:
  `linux-firmware`) **plus every package the enabled repositories offer** whose name matches
  `firmware|microcode|^ucode-` - except the deny list in `vars/main.yml`:
  - packages that download firmware during installation (`facetimehd-firmware`, `b43` installers, ...)
  - packages with an interactive license prompt (Debian `firmware-ipw2x00`, `firmware-ivtv`)
  - boot firmware of other platforms (Raspberry Pi, ARM trusted firmware, Allwinner, Asahi, SpacemiT)
  - firmware bound to an NVIDIA driver version (GSP), tools, -devel/-doc/-debug packages
- **drivers**: kernel module packages (Leap: `kernel-default-extra` with e.g. `intel-vbtn`/`intel-hid` for
  convertibles, and `kernel-default-optional`), video acceleration (VA-API, Vulkan), `iio-sensor-proxy`,
  `libwacom`, `fprintd` (no PAM change), `bluez`, `fwupd`, `iw`. On Fedora the VA-API drivers
  are skipped when the RPM Fusion variant is installed. No NVIDIA driver, no `broadcom-wl` (it disables the
  open Broadcom drivers).
- **guest agents**: `qemu-guest-agent`, `spice-vdagent` - idle on real hardware, active in a VM.

Debian: when `firmware-iwlwifi` is not available, the role stops (non-free-firmware not enabled).

After changes a notice recommends a reboot (new firmware, microcode, kernel modules); initramfs and
microcode updates are done by the package scripts.

## Power management (Leap)

- `power-profiles-daemon` is installed, enabled and started: it provides the power mode switch in GNOME's
  quick settings (power saver / balanced / performance) and sets the CPU energy preference (intel/amd
  pstate EPP) and the laptop's firmware mode (`platform_profile`, e.g. via `ideapad-laptop`/`thinkpad_acpi`)
- `tuned` stays installed (default install) but is disabled and stopped - it changes the same settings and
  would fight power-profiles-daemon (`tuned-ppd` is the alternative provider, not used)
- `thermald` (Intel thermal control) is enabled on Intel CPUs only; on other CPUs it is disabled (it exits
  with "unsupported cpu" there)
- `hardware.power_management: false` skips all of this

## Variables

```yaml
hardware:
  firmware: true          # default
  drivers: true           # default
  guest_agents: true      # default
  power_management: true  # default
  extra_packages: []      # installed in addition
  exclude_packages: []    # exact names removed from all lists
  quirks: []              # hardware quirks for this host, see below
```

## Hardware quirks

Fixes for specific machines from `vars/quirks.yml`, enabled per host (host_vars) by name. The name says what
the fix does; the machine it was needed on is in the catalog comment. Module options can break other
hardware, so enable a quirk only on the machine it is meant for.

| quirk | does | needed on |
|---|---|---|
| `cirrus-cs35l56-17aa3921-spkid1` | links the `-spkid1` CS35L56 firmware to the names the driver requests (woofers play) | Lenovo ThinkBook 16p Gen6 |
| `sof-dmic-num-4` | `options snd_sof_intel_hda_generic dmic_num=4` (4 digital microphones) | Fujitsu LIFEBOOK E5512 |
| `btusb-no-autosuspend` | `options btusb enable_autosuspend=0` (stable Bluetooth) | Lenovo ThinkPad E16 Gen 2 |

```yaml
hardware:
  quirks: [cirrus-cs35l56-17aa3921-spkid1]
```

- firmware links are created in `/usr/lib/firmware` when the target exists (otherwise a notice); they are
  not removed when the quirk is disabled (a leftover link is harmless). If the distribution ships a regular
  file with the link name, it is left alone and a notice says the quirk is not needed anymore.
- module options of all enabled quirks go into `/etc/modprobe.d/50-hardware-quirks.conf`; disabling a quirk
  removes its line, no quirk with module options → the file is removed
- an unknown quirk name stops the run
- takes effect after the next reboot

Finding missing firmware names for a new quirk: `journalctl -k -b | grep 'failed to load'`.

## License

Apache-2.0

Created with the help of AI

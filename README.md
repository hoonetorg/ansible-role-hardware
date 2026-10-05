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
  `libwacom`, `fprintd` (no PAM change), `bluez`, `thermald`, `fwupd`, `iw`. On Fedora the VA-API drivers
  are skipped when the RPM Fusion variant is installed. No NVIDIA driver, no `broadcom-wl` (it disables the
  open Broadcom drivers).
- **guest agents**: `qemu-guest-agent`, `spice-vdagent` - idle on real hardware, active in a VM.

Debian: when `firmware-iwlwifi` is not available, the role stops (non-free-firmware not enabled).

After changes a notice recommends a reboot (new firmware, microcode, kernel modules); initramfs and
microcode updates are done by the package scripts.

## Variables

```yaml
hardware:
  firmware: true          # default
  drivers: true           # default
  guest_agents: true      # default
  extra_packages: []      # installed in addition
  exclude_packages: []    # exact names removed from all lists
```

## License

Apache-2.0

Created with the help of AI

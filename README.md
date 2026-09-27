# WIA3300-10 ImmortalWrt 24.10.6 Clean

Target: Skspruce WIA3300-10 / MediaTek MT7621 / ramips-mt7621.

## Design

This project builds a clean 32MB WIA3300-10 firmware on ImmortalWrt 24.10.6.

The base image includes the device hardware adaptation, LuCI, luci-compat, opkg, USB 3.0/storage foundations, filesystem support, and transparent-proxy kernel foundations.

It deliberately does **not** bundle OpenClash, PassWall, Xray, Mihomo, download services, Samba, or printer services.

## Optional USB profiles

The GitHub Actions workflow provides selectable profiles:

- `clean` — default pure image.
- `usb-storage` — USB flash/HDD/SSD and common filesystem support.
- `usb-printer` — USB printer kernel support.
- `usb-download` — storage/filesystem foundation for network-disk/download use; install the preferred download service separately.
- `usb-extroot` — packages needed to move `/overlay` to USB storage and expand the writable system space.

The profiles are configuration fragments under `config/profiles/`. They are optional and are not included in the default clean build.

## Proxy readiness

The clean image keeps the relevant nft/socket/tproxy kernel modules available as a foundation for later user-installed PassWall/OpenClash-type software, without including those software packages themselves.

## Hardware

- MT7621AT
- 32MB NOR
- DSA Ethernet
- LAN1-LAN4 + WAN
- MT7615 2.4G/5G
- USB 3.0
- WIA3300-10 factory MAC handling

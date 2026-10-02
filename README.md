<img width="1319" height="1794" alt="mm_facetoface_collect_qrcode_1790965343705" src="https://github.com/user-attachments/assets/8d0f1de4-3071-4445-aa62-fe50124d89b6" />
<img width="1080" height="1620" alt="1790965237559" src="https://github.com/user-attachments/assets/8388122e-1183-4e68-819b-59b26836820e" />
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
项目支持

本项目为开源项目。

WIA3300-10 的硬件适配、固件编译、实机测试、问题排查和持续维护需要投入时间与精力。

如果本项目对你有帮助，欢迎自愿支持项目后续维护。

支持作者

支付宝![支付宝](这里放支付![支付宝](alipay.png)图片文件名)

微信支付![微信支付](这里放微信![微信支付](wechat.png)文件名)

赞助完全自愿，不影响固件使用、下载或源码获取。

# WIA3300-10 ImmortalWrt 24.10.6 Clean

This repository builds a clean ImmortalWrt 24.10.6 image for Skspruce WIA3300-10.

Hardware target:
- MediaTek MT7621AT
- ramips/mt7621
- 32 MiB SPI-NOR
- 256 MiB RAM
- WIA3300-10 board-specific DTS, partitions, Ethernet, LEDs, reset and USB

Package policy:
- LuCI
- luci-compat
- opkg
- USB storage and USB 3 support
- nftables/TProxy prerequisites
- no bundled OpenClash
- no bundled PassWall
- no Xray
- no sing-box
- no Mihomo

# WIA3300-10 天灵纯净版 · ImmortalWrt 24.10.6

> 为 WIA3300-10 打造的 **ImmortalWrt 24.10.6 长期维护型纯净固件**。

[![ImmortalWrt](https://img.shields.io/badge/ImmortalWrt-24.10.6-blue)](https://github.com/immortalwrt/immortalwrt)
[![Target](https://img.shields.io/badge/Target-WIA3300--10-orange)](https://github.com/zhzmst2020/WIA3300-10-ImmortalWrt-24.10.6-Clean)
[![Platform](https://img.shields.io/badge/Platform-MT7621-green)](https://github.com/zhzmst2020/WIA3300-10-ImmortalWrt-24.10.6-Clean)

这是一个面向 **Skspruce WIA3300-10 / MediaTek MT7621** 的独立适配项目。

项目目标不是制作一个“什么都塞进去”的大杂烩固件，而是建立一个**稳定、干净、可扩展、适合长期维护的 24.10.6 基础系统**。

---

## 项目理念

### 一版扎实，后续以 Bug 修复和维护为主

WIA3300-10 的硬件适配、固件编译、实机刷写、USB/存储、Extroot、软件兼容性等均经过实际测试。

当前项目已经完成基本适配，后续重点放在：

- 已知问题修复
- 硬件兼容性维护
- USB / 存储 / Extroot 稳定性
- LuCI 与系统基础功能
- 第三方软件安装环境兼容性
- ImmortalWrt 24.10.6 长期维护

不追求为了“更新版本号”而频繁改变架构。

---

## 为什么选择 24.10.6？

本项目选择 ImmortalWrt **24.10.6** 作为当前长期维护基础。

原因很简单：

- 系统成熟度较高
- MT7621 生态相对稳定
- opkg 软件包体系完整
- 适合与外置存储结合扩展
- 可以保持“固件与软件分离”的设计

项目不以集成大量第三方插件为目标。

---

## 纯净版：纯净，而不是简陋

默认固件**不内置**：

- OpenClash
- PassWall
- Xray
- Mihomo / Clash 内核
- 下载服务
- Samba
- USB 打印服务

这样做是为了避免把特定用户需求强行写入基础固件，也尽量节省 WIA3300-10 有限的 32MB 闪存空间。

用户可以根据自己的需求，在刷机后自行安装所需软件。

---

## 代理软件运行环境

虽然不预装第三方代理软件，但基础系统预留了后续软件运行所需要的环境。

重点包括：

- luci-compat
- dnsmasq-full
- bash
- curl
- ca-bundle
- ip-full
- kmod-tun
- nft/socket/tproxy 相关内核模块

设计原则：

> **固件负责打好基础，软件由用户按需选择。**

这样既保持系统纯净，也避免因为某一个插件的版本变化而频繁重新制作基础固件。

---

## USB 与 Extroot 扩展

项目针对 WIA3300-10 的 USB 扩展进行了实际测试。

支持方向包括：

- USB Storage
- ext4 / vfat 等常用文件系统
- 外置 Overlay
- Extroot
- 外置软件包和插件空间

### U盘扩展（Extroot）

WIA3300-10 推荐使用 **ImmortalWrt 原生 Extroot** 扩展存储。

> ⚠️ **注意：** 以下操作会格式化 U 盘，U 盘中的原有数据将被清除。请确认 U 盘设备节点后再操作。

#### 1. 安装扩展所需软件

SSH 登录路由器后执行：

```sh
opkg update
opkg install block-mount kmod-usb-storage kmod-fs-ext4 e2fsprogs kmod-fs-vfat
```

#### 2. 查看 U 盘设备

执行：

```sh
block info
```

确认 U 盘分区，例如：

```text
/dev/sda1
```

以下步骤以 `/dev/sda1` 为例。

#### 3. 格式化 U 盘

先卸载 U 盘分区：

```sh
umount /dev/sda1
```

格式化为 ext4：

```sh
mkfs.ext4 /dev/sda1
```

格式化完成后重新查看 UUID：

```sh
block info
```

找到 `/dev/sda1` 对应的 UUID，并记录下来。

例如：

```text
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

#### 4. 临时挂载 U 盘

创建临时挂载目录：

```sh
mkdir /mnt/extroot
```

挂载 U 盘：

```sh
mount /dev/sda1 /mnt/extroot
```

确认挂载成功：

```sh
df -h
```

应该能够看到 `/mnt/extroot`。

#### 5. 复制当前 Overlay 数据

将当前系统的可写层复制到 U 盘：

```sh
tar -C /overlay -cvf - . | tar -C /mnt/extroot -xf -
```

> 这里只复制当前 `/overlay` 数据，不复制整个 `/rom` 固件系统。

#### 6. 配置 Extroot

删除旧的 Extroot 配置：

```sh
uci -q delete fstab.extroot
```

创建新的挂载配置：

```sh
uci set fstab.extroot='mount'
uci set fstab.extroot.uuid='你的U盘UUID'
uci set fstab.extroot.target='/overlay'
uci set fstab.extroot.enabled='1'
uci commit fstab
```

将 `你的U盘UUID` 替换成第 3 步查询到的实际 UUID。

#### 7. 重启路由器

```sh
reboot
```

等待路由器重新启动后，重新进入 LuCI 管理页面。

#### 8. 在 LuCI「挂载点」中添加挂载点

进入：

**系统 → 挂载点**

在挂载点页面添加 U 盘挂载点。

选择第 3 步记录的 U 盘 UUID，挂载点设置为：

```text
/overlay
```

启用该挂载点并保存、应用配置。

> 如果页面中已经存在对应的 `/overlay` 挂载配置，则检查 UUID、挂载点和启用状态即可，不需要重复创建。

完成挂载点配置并应用后，Extroot 即完成。

此时可直接在 LuCI 页面查看可用存储空间，确认软件包/可写空间已经转移到 U 盘。

> **本项目推荐使用 ImmortalWrt 原生 Extroot，不需要将整个根文件系统复制到 U 盘。**

### GitHub Actions 扩展配置

GitHub Actions 提供可选配置：

| Profile | 用途 |
|---|---|
| clean | 默认纯净基础固件 |
| usb-storage | USB 存储及常用文件系统 |
| usb-printer | USB 打印机支持 |
| usb-download | 下载/网络存储基础环境 |
| usb-extroot | Extroot 所需软件包 |

这些 Profile 位于 config/profiles/，默认不会全部加入基础固件。

---

## 硬件规格

| 项目 | 规格 |
|---|---|
| SoC | MediaTek MT7621AT |
| Flash | 32MB NOR |
| RAM | 256MB |
| Ethernet | DSA |
| Ports | LAN1 / LAN2 / LAN3 / LAN4 / WAN |
| Wi-Fi | MediaTek MT7615 2.4GHz / 5GHz |
| USB | USB 2.0 |
| Target | ramips/mt7621 |
| Device | skspruce_wia3300-10 |

---

## 固件编译

项目使用 GitHub Actions 自动编译。

源码基础：
- ImmortalWrt 24.10.6
- Target：ramips/mt7621
- Device：skspruce_wia3300-10

工作流：

**Build WIA3300-10 ImmortalWrt 24.10.6 Clean**

可以通过 Actions 选择不同 Profile 进行构建。

---

## 固件与源码

本项目所有核心代码、构建配置和补丁均公开。

你可以：
1. 查看源码
2. 查看 GitHub Actions 编译过程
3. 下载构建完成的固件
4. 根据自己的需求修改配置
5. 提交 Issue 反馈问题

项目地址：

https://github.com/zhzmst2020/WIA3300-10-ImmortalWrt-24.10.6-Clean

---

## 项目支持 ❤️

这是一个由个人长期投入时间维护的开源项目。

从硬件适配、源码修改，到 GitHub Actions 编译，再到实际刷机、USB 扩展、Extroot、软件安装和各种异常排查，都需要持续投入时间和测试成本。

一个固件文件看起来很小，但它背后往往经历：

> 编译 → 刷机 → 测试 → 发现问题 → 排查 → 修改 → 再编译 → 再刷机 → 再测试

如果这个项目对你有帮助，欢迎通过**资金赞助**支持后续开发与维护。

赞助金额不限，**金额多少并不重要，支持本身就是对项目的一种认可。**

### 支持作者

**微信支付**

![微信支付](mm_facetoface_collect_qrcode_1790965343705.png)

**支付宝**

![支付宝](1790965237559.jpg)

> 赞助完全自愿。
>
> 不影响固件使用、下载、源码获取，也不会因为是否赞助而限制项目功能。

感谢每一位使用、测试、反馈和支持这个项目的人。

---

## 免责声明

本项目为个人独立维护项目，与 ImmortalWrt 官方项目不存在隶属关系。

ImmortalWrt 本身是开源项目，本项目基于其开源代码进行 WIA3300-10 的设备适配和定制。

使用本固件前请确认设备型号和刷机方式。刷机存在风险，请自行承担相关风险。

---

## 项目进度

- [x] WIA3300-10 硬件适配
- [x] ImmortalWrt 24.10.6 基础系统
- [x] DSA 网络适配
- [x] USB Storage 基础支持
- [x] Extroot 环境
- [x] Proxy Ready 基础环境
- [x] GitHub Actions 自动编译
- [x] 多台实机测试
- [ ] 持续收集问题与反馈
- [ ] 后续 Bug 修复
- [ ] 长期持续维护

> **目标不是做一个功能最多的固件，而是做一个真正稳定、干净、可长期使用的 WIA3300-10 24.10.6 基础固件。**
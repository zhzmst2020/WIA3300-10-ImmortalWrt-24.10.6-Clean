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
- Extroot
- 外置软件包和插件空间

### U盘扩展（Extroot）

WIA3300-10 使用 ImmortalWrt 原生 Extroot，将外置 ext4 分区作为新的可写 Overlay。固件内提供 `/usr/sbin/wia3300-extroot-setup` 辅助脚本，用于校验分区、复制现有 Overlay 并生成配置，避免在 SSH 交互窗口中逐条粘贴容易中断的命令。

> **重要：格式化会清空指定分区。请先确认设备名，并备份 U 盘中需要保留的数据。辅助脚本本身不会格式化 U 盘。**

#### 1. 确认 U 盘分区

插入 U 盘后执行：

```sh
block info
```

确认目标分区，例如 `/dev/sda1`，并确认它不是其他磁盘或重要数据分区。不要仅凭示例照抄设备名。

#### 2. 格式化为 ext4（破坏性操作）

只有在确认分区可清空后，才执行以下命令。将设备名替换为上一步确认的分区：

```sh
mkfs.ext4 /dev/sda1
```

该命令会删除该分区原有文件。若命令报错，先停止并检查原因；不要把格式化和后续步骤写成一条带 `exit 1` 的长命令，以免错误处理直接结束当前 SSH shell。

格式化完成后确认 UUID 和文件系统类型：

```sh
block info /dev/sda1
```

输出应包含 `UUID="..."` 和 `TYPE="ext4"`。

#### 3. 执行固件自带的 Extroot 设置脚本

```sh
/usr/sbin/wia3300-extroot-setup /dev/sda1
```

脚本会执行以下检查和操作：

- 确认参数是块设备，且 `block info` 能读取有效 UUID。
- 只接受 ext4 分区；**不会自行格式化分区**。
- 如果设备已经挂载到活动的 `/overlay`，会拒绝继续，避免覆盖正在使用的 extroot。
- 检查目标分区是否为空（允许 `lost+found`）；发现其他文件时停止，不擅自删除。
- 先创建归档，再解压复制，并输出阶段提示；复制成功后才写入 `fstab` 配置。
- 将 `fstab.extroot` 设置为该分区 UUID 和 `/overlay`，并把启动等待时间设为 15 秒，以降低 USB 设备启动较慢导致的漏挂载风险。

如果脚本报告错误，请保留完整报错并停止，不要反复格式化或重启碰运气。

#### 4. 重启并验证

脚本成功完成并显示已保存的 Extroot 配置后，再执行：

```sh
reboot
```

重新连接后执行：

```sh
mount | grep -E '(/overlay|/dev/sda1)'
df -h / /overlay
```

成功时，实际 USB ext4 分区应挂载到 `/overlay`，而且 `/` 的可用空间应与外置分区相对应。

#### 5. 如果重启后仍没有使用 U 盘

先不要重复格式化。执行以下只读检查并保存输出：

```sh
block info
uci show fstab
logread | sed -n -e "/- preinit -/,/- init -/p"
```

检查重点是启动早期的 extroot 日志、UUID 是否匹配，以及 USB 分区是否在 preinit 阶段及时出现。请依据日志定位，不要在没有证据时反复修改挂载配置。

> 注意：本方案使用 ext4 扩展 `/overlay`，不使用 Swap，也不把整个系统搬到 U 盘。若拔掉 U 盘，系统可能退回内部闪存中的原有 overlay；不要在外置 overlay 已启用时随意升级内核或内核模块。

## 插件与内核安装

Extroot 完成后，可以根据需要安装 OpenClash 和 PassWall。

> **本节记录的是本项目在 WIA3300-10 + ImmortalWrt 24.10.6 环境中实际验证过的安装方式。不同版本软件的页面和安装方式可能有所变化。**

### OpenClash

1. 在 LuCI 页面安装 **OpenClash** 本体，可按实际页面提供的方式上传软件包或从软件源安装。
2. OpenClash 本体安装完成后，**不需要重启路由器**。
3. 退出当前 LuCI 登录页面，刷新浏览器后重新登录路由器后台。
4. 此时 OpenClash 菜单即可正常出现。
5. 进入 **服务 → OpenClash**，按页面提示安装 **Meta 内核**。

> OpenClash 的 Meta 内核直接按页面提示安装即可，不需要通过 SSH 手动安装。

### PassWall

#### PassWall 本体

PassWall 本体可以直接在 LuCI 页面安装：

- 上传对应的 `.ipk` 软件包安装；或
- 从软件源搜索并安装。

安装完成后进入 **服务 → PassWall** 即可。

#### PassWall 内核

PassWall 的 Xray、Sing-box、Hysteria 等核心，**不要依赖 PassWall 页面中的“软件更新”来安装**。

在本项目当前环境中，推荐通过 **SSH** 安装所需核心。

例如：

```sh
opkg update
opkg install xray-core
```

```sh
opkg install sing-box
```

```sh
opkg install hysteria
```

Geoview 如有需要，也可通过 SSH 安装：

```sh
opkg install geoview
```

实际使用时**缺哪个安装哪个，已经安装的核心不要重复安装**。

> **重要：** PassWall 页面中的内核更新失败，不代表固件无法安装或运行这些核心。当前环境实测通过 SSH 安装核心可以正常使用。

> **不要为了更新核心而随意更换本项目的软件源，也不要在没有明确缺失依赖证据的情况下批量补装依赖。** 本项目固件已经预置相关基础运行环境，正常情况下应保持软件源和系统环境稳定。

---

### 推荐安装组合

对于 WIA3300-10 的 256MB RAM + 32MB Flash 平台，本项目已经实际测试过以下组合：

- OpenClash
- OpenClash Meta 内核
- PassWall
- Geoview
- Xray
- Sing-box
- Hysteria

在使用 256M U 盘进行 Extroot 后，上述组合仍保留了较充足的可用空间。

> **扩容不是越大越好，而是看设备实际需要多少。** 对 WIA3300-10 而言，256M U 盘已经能够满足这套软件与核心组合的实际使用需求；更大的 U 盘可以留给更适合大容量扩展的设备。

> 本项目教程的目的，是把已经实际踩过并解决的问题记录下来，尽量避免后来使用者重复踩坑。

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

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

> ⚠️ **重要：以下步骤会格式化 U 盘，U 盘中的原有数据将被清除。请先确认 U 盘设备节点。**
>
> **本教程只适用于标准的 SquashFS + JFFS2 Overlay 环境。不要同时使用其他 OverlayFS/upper/work 教程。**

#### 1. 安装扩展所需软件

SSH 登录路由器后执行：

```sh
opkg update
opkg install block-mount kmod-usb-storage kmod-fs-ext4 e2fsprogs
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

**下面所有命令均以 `/dev/sda1` 为例。你的设备节点不同就必须替换。**

#### 3. 格式化 U 盘

先卸载：

```sh
umount /dev/sda1
```

格式化为 ext4：

```sh
mkfs.ext4 /dev/sda1
```

格式化完成后重新查询 UUID：

```sh
block info
```

找到 `/dev/sda1` 对应的实际 UUID。

#### 4. 临时挂载 U 盘

创建临时目录并挂载：

```sh
mkdir -p /mnt/extroot
mount /dev/sda1 /mnt/extroot
```

确认：

```sh
df -h /mnt/extroot
```

#### 5. 复制当前 Overlay 数据

**这一步必须在配置 Extroot 之前完成。**

执行：

```sh
tar -C /overlay -cvf - . | tar -C /mnt/extroot -xf -
```

复制完成后检查：

```sh
ls -la /mnt/extroot
```

应该能看到当前 Overlay 中的配置文件和目录。

#### 6. 自动写入 Extroot 配置（不要手抄 UUID）

**不要把“你的U盘UUID”之类的占位文字直接写进 `/etc/config/fstab`。**

先自动读取 `/dev/sda1` 的 UUID：

```sh
UUID="$(block info /dev/sda1 | sed -n 's/.*UUID="\\([^"]*\\)".*/\\1/p')"
```

检查：

```sh
echo "$UUID"
```

必须显示真实 UUID。如果没有任何输出，**停止操作，不要继续重启**。

确认 UUID 正确后，执行：

```sh
uci -q delete fstab.extroot
uci set fstab.extroot='mount'
uci set fstab.extroot.uuid="$UUID"
uci set fstab.extroot.target='/overlay'
uci set fstab.extroot.enabled='1'
uci commit fstab
```

检查配置：

```sh
uci show fstab.extroot
```

结果中的 `uuid` 必须是刚才查询到的真实 UUID，而不是“你的U盘UUID”。

#### 7. 解除临时挂载并重启

先卸载临时挂载：

```sh
umount /mnt/extroot
```

然后重启：

```sh
reboot
```

#### 8. 重启后验证 Extroot 是否成功

路由器重新启动并 SSH 登录后，执行：

```sh
df -h
```

**成功的判断标准：**

- `/` 不再只有内部 Flash 的约 19MB Overlay 容量；
- `/overlay` 应该由 U 盘的 ext4 分区提供；
- U 盘容量应明显出现在 `/` 或 `/overlay` 的可用空间中。

再执行：

```sh
mount
```

确认 U 盘已经作为 Overlay 的底层存储参与系统根文件系统。

如果重启后仍然看到：

```text
/dev/mtdblock6 ... /overlay
/dev/sda1 ... /mnt/sda1
```

而不是 U 盘参与 Overlay，说明 Extroot 没有成功，**不要继续重复执行配置命令**，先检查启动日志和 fstab。

> **特别注意：不要在 Extroot 配置完成后，再在 LuCI「系统 → 挂载点」里重复创建一个 `/overlay` 挂载点。**
>
> `/overlay` 的 Extroot 配置已经由 `/etc/config/fstab` 完成。重复添加挂载可能造成冲突。

#### 9. 关于 `upper` / `work`

本项目的标准 Extroot 教程**不要求手动创建**：

```text
/mnt/sda1/upper
/mnt/sda1/work
```

也不要把“创建 upper/work 并手动拼接 OverlayFS”与本教程的原生 Extroot 方案混用。

> **核心原则：一个 U 盘只使用一种 Overlay 扩展方案。**
>
> 本项目选择的是 ImmortalWrt 原生 Extroot：**复制当前 `/overlay` → U 盘 → `/etc/config/fstab` 指向 U 盘 UUID 和 `/overlay` → 重启验证。**

---

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

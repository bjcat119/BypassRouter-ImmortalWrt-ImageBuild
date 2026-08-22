[中文版](README.md) | [English Version](README_EN.md)

# 🚀 BypassRouter-ImmortalWrt-ImageBuild

<p align="center">
  <a href="https://github.com/bjcat119/BypassRouter-ImmortalWrt-ImageBuild/actions"><img src="https://img.shields.io/badge/Automation-GitHub%20Actions-2088FF?logo=githubactions&logoColor=white" alt="GitHub Actions"></a>
  <a href="https://downloads.immortalwrt.org/releases/24.10.6"><img src="https://img.shields.io/badge/ImmortalWrt-24.10.6-success" alt="ImmortalWrt 24.10.6"></a>
  <a href="https://github.com/bjcat119/BypassRouter-ImmortalWrt-ImageBuild/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="MIT License"></a>
  <a href="https://github.com/bjcat119/BypassRouter-ImmortalWrt-ImageBuild"><img src="https://img.shields.io/badge/Arch-mvebu%2Fcortexa9-orange" alt="mvebu/cortexa9"></a>
  <a href="https://github.com/bjcat119/BypassRouter-ImmortalWrt-ImageBuild"><img src="https://img.shields.io/badge/Target-linksys__wrt3200acm-important" alt="WRT3200ACM"></a>
</p>

<p align="center">
  <b>基于 GitHub Actions + ImmortalWrt Image Builder 自动构建 Linksys WRT3200ACM（Rango）旁路由固件</b><br>
  <span>刷上即处于<b>旁路由（透明网关）</b>状态：LAN 静态 <code>192.168.1.2</code>、关 DHCP、关无线、防火墙 LAN→ACCEPT + WAN MASQUERADE；Luci 已预装 <b>Homeproxy + sing-box TUN</b>，贴节点即可让全屋设备访问 ChatGPT / Gemini / Claude / Copilot。</span>
</p>

---

## ✨ 特性一览

- 🔌 **开机即用旁路由**：`uci-custom` 首启脚本自动固化 LAN IP、关 DHCP/无线、配防火墙，无需刷后手动改配置。
- 🌐 **Homeproxy + sing-box TUN**：Luci 可视化配置，TCP/UDP 全接管，Gemini / Claude / Copilot 透明代理。
- 🛡️ **双分区救砖**：预装 `luci-app-advanced-reboot`，刷坏一键切分区，WRT3200ACM 双 UBI 分区更安全。
- 💽 **USB 存储全格式**：`block-mount` + `blockd` + ext4/vfat/ntfs/exfat，U 盘即插即用。
- 🎨 **Argon 主题 + 中文**：`luci-theme-argon` + 全套 `luci-i18n-*-zh-cn`，后台体验友好。
- ⚙️ **GitHub Actions 自动构建**：Fork / Use this template 后一键出 `sysupgrade.bin` + `factory.img`。

---

## 📟 硬件与目标

| 项目 | 详情 |
|---|---|
| **设备** | Linksys WRT3200ACM v1（Rango） |
| **SoC** | Marvell Armada 385 88F6820（Cortex-A9） |
| **架构** | `mvebu/cortexa9`，包架构 `arm_cortex-a9_vfpv3-d16` |
| **无线** | Marvell 88W8964（mwlwifi），旁路由场景默认关闭 |
| **Flash / RAM** | 256MB NAND / 512MB DDR3 |
| **Image Builder PROFILE** | `linksys_wrt3200acm` |


---

## 📁 仓库结构

```
BypassRouter-ImmortalWrt-ImageBuild/
├── .github/workflows/image-builder.yml   # Image Builder 构建流
├── packages.list                         # 包清单（Homeproxy / sing-box / kmod-tun …）
├── uci-custom                           # 首启脚本 → /etc/uci-defaults/99-custom
├── packages/                            # 可选：额外 .ipk（架构须为 arm_cortex-a9_vfpv3-d16）
├── README.md                            # 本文件（中文）
└── README_EN.md                         # English README
```

---

## 📦 预装软件

### 网络核心（替换默认）

| 包 | 说明 |
|---|---|
| `dnsmasq-full` | 替换默认 dnsmasq，支持 DNSSEC / Nftables |
| `wpad-openssl` | 替换 wpad-basic，支持 WPA3 / 802.11r |
| `ip-full` | 完整 iproute2 工具集 |
| `firewall4` | fw4 / nftables 防火墙 |

### 旁路由代理（Homeproxy TUN）

| 包 | 说明 |
|---|---|
| `luci-app-homeproxy` | Homeproxy Luci 前端（含配置生成） |
| `luci-i18n-homeproxy-zh-cn` | Homeproxy 中文翻译 |
| `sing-box` | 代理引擎（TUN 由 sing-box 创建） |
| `kmod-tun` | TUN 字符设备 |
| `kmod-nft-tproxy` | TPROXY / 重定向依赖 |


### 管理与主题

| 包 | 说明 |
|---|---|
| `luci-light` + 中文翻译 | 精简 LuCI 核心 |
| `luci-theme-argon` | Argon 主题 |
| `luci-app-package-manager` | 网页包管理器 |
| `luci-app-advanced-reboot` | 双分区切换 / 刷 factory（救砖神器） |
| `luci-app-autoreboot` | 定时重启 |
| `autocore` | CPU 频率 / 温度状态 |

### 存储 & USB

| 包 | 说明 |
|---|---|
| `automount` | 分区热插拔自动挂载 |
| `kmod-usb3` / `kmod-usb-storage` / `kmod-usb-storage-uas` | USB 3.0 存储驱动 |
| `kmod-fs-ext4` / `kmod-fs-vfat` / `kmod-fs-ntfs` / `kmod-fs-exfat` | 全格式文件系统 |
| `e2fsprogs` | ext 分区工具 |


---

## 🔧 构建步骤

### 1️⃣ 创建你的构建仓库

右上角 **Use this template → Create a new repository**，在自己的账号下开出新仓。

### 2️⃣ 自定义（可选）

| 文件 | 用途 |
|---|---|
| `packages.list` | 增删包，一行一个；`-包名` 剔除默认包 |
| `uci-custom` | 首启脚本（改 LAN IP / 主路由 IP / root 密码等） |
| `packages/` | 放额外 `.ipk`（架构必须是 `arm_cortex-a9_vfpv3-d16`） |

### 3️⃣ 触发构建

进入 **Actions → ImmortalWrt Image Builder → Run workflow**，PROFILE 默认 `linksys_wrt3200acm`，等待约 3–8 分钟。

### 4️⃣ 下载固件

在 Artifacts 区域下载 `linksys_wrt3200acm/` 目录，内含：

| 文件 | 用途 |
|---|---|
| `*-squashfs-sysupgrade.bin` | 从 OpenWrt / ImmortalWrt 升级 |
| `*-squashfs-factory.img` | 从 Linksys 原厂固件刷入 |

---

## 🔌 刷机与接线

- **接线**：WRT3200ACM **只插 LAN 口**接主路由 LAN 口，**WAN 口空置**。
- **双分区保护**：开机按住电源键 3 秒切换分区，或通过 **Luci → 系统 → 高级重启** 操作。
- **首启自动化**：`uci-custom` 首次启动自动执行 —— LAN `192.168.1.2`、网关/DNS 指主路由 `192.168.1.1`、关 DHCP、关 radio0/1。

---

## 📱 客户端如何走旁路

**方式 A · 手动指定**（单台设备）：
- IP 同网段（如 `192.168.1.x`）
- 网关 `192.168.1.2`
- DNS `192.168.1.2`

**方式 B · 全屋自动**（主路由 DHCP 下发）：
- 主路由 DHCP 的“默认网关”与“DNS”均填 `192.168.1.2`
- 华为星光等原厂固件若只让改 DNS，则 DNS 填 `192.168.1.2` + 终端手动网关

> ✅ 客户端**无需**安装任何代理 App（Shadowrocket / Clash / v2rayNG 等），流量在网络层被 TUN 接管。

---

## 🛡️ Homeproxy TUN 首次配置

进入 **Luci → 服务 → HomeProxy**：

1. **节点设置**：粘贴订阅链接，或手动添加 VLESS / Trojan / Hysteria2 节点并选择 Main Node。
2. **客户端设置**：
   - 代理模式 = **TUN**（TCP + UDP 全进 `singtun0`）
   - 路由模式：先选 `global` 测通，再换 `bypass_mainland_china`
   - DNS 服务器：`8.8.8.8`（国际），中国 DNS 留 `223.5.5.5`
3. **启用** Homeproxy → 状态页显示 **RUNNING**、节点延迟正常。
4. **验证**：浏览器打开 `gemini.google.com` / `claude.ai` / `github.com/copilot`。

> 🔥 **防火墙无需手动配置**：Homeproxy 会自动写入 fw4 include，`singtun0` 由 sing-box 自建，旁路由 MASQUERADE 已在 `uci-custom` 中固化。Luci → 状态 → 防火墙 看到 `singbox` / `homeproxy` 相关 nft chain 即表示注入生效。

---

## ⚠️ 注意事项

- 📡 **无线驱动**：mwlwifi 在 88W8964 上 5G 部分客户端存在兼容性波动，旁路由关无线不影响使用。
- 💪 **性能预期**：WRT3200ACM 为 ARMv7 软路由，sing-box TUN 跑 Gemini / Claude / Copilot 文字流量绰绰有余；**不建议叠加满速 4K 全局代理**。
- 💽 **NTFS 读写**：使用内核 `kmod-fs-ntfs`（ntfs3 驱动），无需 ntfs-3g；个别老 UAS 硬盘盒不兼容时，在启动参数加 `usb-storage.quirks=xxxx:yyyy:u`。
- 🌍 **节点地区**：Claude 对部分地区 IP 会提示 “App unavailable in your region”，建议选美西 / 日本 / 新加坡等干净机房节点，避免香港节点跑 Claude。
- 🔄 **双分区救砖**：刷坏无需紧张，电源键 3 秒切分区即可恢复。

---

## 🔗 相关链接

- [ImmortalWrt 官网](https://immortalwrt.org)
- [ImmortalWrt 24.10.6 mvebu/cortexa9 下载](https://downloads.immortalwrt.org/releases/24.10.6/targets/mvebu/cortexa9/)
- [OpenWrt 设备页（WRT3200ACM）](https://openwrt.org/toh/linksys/wrt3200acm)

> 固件版权归 [ImmortalWrt 项目](https://github.com/immortalwrt/immortalwrt) 所有。本仓库仅提供构建配置。

<p align="center">
  <sub>MIT License · © bjcat119</sub>
</p>



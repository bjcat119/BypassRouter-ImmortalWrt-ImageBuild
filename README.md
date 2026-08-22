[🇨🇳 中文版](README.md) | [🇺🇸 English Version](README_EN.md)

# ImmortalWrt 24.10.6 for Linksys WRT3200ACM 旁路由

[![ImmortalWrt](https://img.shields.io/badge/ImmortalWrt-24.10.6-brightgreen)](https://immortalwrt.org/)
[![Target](https://img.shields.io/badge/target-mvebu%2Fcortexa9-blue)](https://downloads.immortalwrt.org/releases/24.10.6/targets/mvebu/cortexa9/)
[![PROFILE](https://img.shields.io/badge/profile-linksys_wrt3200acm-orange)](https://openwrt.org/toh/linksys/wrt3200acm)

基于 GitHub Actions + ImmortalWrt Image Builder​ 自动构建 Linksys WRT3200ACM（Rango）​ 旁路由固件（ImmortalWrt 24.10.6 / mvebu/cortexa9）。

刷上即处于旁路由（透明网关）状态：LAN 静态 192.168.1.2、关 DHCP、关无线、防火墙 LAN→ACCEPT + WAN MASQUERADE；Luci 已装 Homeproxy + sing-box TUN，贴节点即可让全屋设备访问 Gemini / Claude / Copilot。

---

## 硬件信息

| 项目 | 详情 |
|------|------|
| 设备 | 	Linksys WRT3200ACM （Rango） |
| SoC | Marvell Armada 385 88F6820 (Cortex-A9) |
| 无线 | Marvell 88W8964（mwlwifi），旁路由场景默认关闭 |
| Flash | 256MB NAND |
| RAM | 512MB DDR3 |
| 架构 | `mvebu/cortexa9` |
| PROFILE | `linksys_wrt3200acm` |

---

## 预装软件

### 网络核心（替换默认）
| 包 | 说明 |
|----|------|
| `dnsmasq-full` | 替换默认 dnsmasq，支持 DNSSEC / Nftables |
| `wpad-openssl` | 替换 wpad-basic，支持 WPA3 / 802.11r |
| `ip-full` | 完整 iproute2 工具集 |

### LuCI & 主题
| 包 | 说明 |
|----|------|
| `luci-light` + 中文翻译 | 精简 LuCI 核心 |
| `luci-theme-argon` | Argon 主题 |
| `luci-app-package-manager` | 网页包管理器 |

### 系统管理
| 包 | 说明 |
|----|------|
| `luci-app-advanced-reboot` | 双分区切换 / 刷 factory（Linksys 救砖神器） |
| `luci-app-autoreboot` | 定时重启 |
| `luci-app-partexp` | 网页分区扩容（一键扩展 rootfs） |
| `luci-app-upnp` | UPnP / NAT-PMP |
| `autocore` | CPU 频率 / 温度等状态信息 |
| `bash` `curl` `wget` `ca-certificates` `ca-bundle` | 基础工具 |

### 存储 & USB
| 包 | 说明 |
|----|------|
| `automount` + `block-mount` | U 盘 / 移动硬盘自动挂载 |
| `kmod-usb3` `kmod-usb-storage` `kmod-usb-storage-uas` | USB 3.0 存储驱动 |
| `kmod-fs-ext4` `kmod-fs-vfat` `kmod-fs-ntfs3` `kmod-fs-exfat` | 全格式文件系统支持 |
| `e2fsprogs` | ext 分区工具 |

### 无线驱动
| 包 | 说明 |
|----|------|
| `kmod-mwlwifi` | mwlwifi 驱动（Marvell 88W8864） |
| `mwlwifi-firmware-88w8864` | 对应固件 |
| `iwinfo` | 无线信息工具 |

---

## 仓库结构
├── .github/workflows/

│   └── image-builder.yml      # 构建流程

├── packages.list              # 软件包清单（本文件）

├── uci-custom                 # 首次启动脚本 → /etc/uci-defaults/99-custom

├── packages/                  # 可选：额外 .ipk（需 arm_cortex-a9_neon 架构）

├── README.md

└── README_EN.md

---

## 构建步骤

### 1. Fork 本仓库
右上角 **Use this template → Create a new repository**。

### 2. 自定义（可选）
| 文件 | 用途 |
|------|------|
| `packages.list` | 增删包，一行一个，`-包名` 删除默认包 |
| `uci-custom` | 首启脚本（改 LAN IP / 设root密码 / 设置PPPoE拨号账号密码） |
| `packages/` | 放额外 `.ipk`（架构必须是 `arm_cortex-a9_neon`） |

### 3. 触发构建
- **Actions** → **ImmortalWrt Image Builder** → **Run workflow**
- `Target PROFILE`：默认 `linksys_wrt1900ac-v2`
- 等待 3–8 分钟

### 4. 下载固件
Artifacts 区域下载 `linksys_wrt1900ac-v2/` 目录，内含：

| 文件 | 用途 |
|------|------|
| `*-squashfs-sysupgrade.bin` | 从 OpenWrt / ImmortalWrt 升级 |
| `*-squashfs-factory.img` | 从 Linksys 原厂固件刷入 |

---

## 刷机注意事项

- ⚠️ **PROFILE 名称**：24.10 系列为 `linksys_wrt1900ac-v2`（带横杠），旧名 `linksys_wrt1900acv2` 会构建失败。
- ⚠️ **双分区保护**：WRT1900AC v2 有双 boot 分区，刷坏可在开机时按住 **电源键 3 秒** 切换分区，或通过 `advanced-reboot` 网页操作。
- ⚠️ **无线驱动**：mwlwifi 驱动稳定性不如 ath79 平台，5G 部分客户端可能有兼容问题。
- ⚠️ **NTFS 读写**：使用内核 `ntfs3` 驱动，无需 `ntfs-3g`；极少数老硬盘盒 UAS 不兼容时，需在启动参数加 `usb-storage.quirks`。
- ⚠️ **首次启动后扩容**：LuCI → 系统 → 分区扩容，一键扩容路由器的/overlay空间到外接的USB磁盘上。

---

## 相关链接

- [ImmortalWrt 官网](https://immortalwrt.org/)
- [24.10.6 mvebu/cortexa9 下载](https://downloads.immortalwrt.org/releases/24.10.6/targets/mvebu/cortexa9/)
- [OpenWrt 设备页](https://openwrt.org/toh/linksys/wrt1900ac_v2)
- [原模板 noviachen/Image-Builder](https://github.com/noviachen/Image-Builder)

---

*固件版权归 ImmortalWrt 项目所有。本仓库仅提供构建配置。*

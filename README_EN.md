[中文版](README.md) | [English Version](README_EN.md)


🚀 **BypassRouter-ImmortalWrt-ImageBuild**

Auto-build **Linksys WRT3200ACM (Rango)** bypass-router firmware via **GitHub Actions + ImmortalWrt Image Builder** (ImmortalWrt 24.10.6 / mvebu/cortexa9).

Flashed as-is, the device boots into a **bypass gateway (transparent proxy)** state: static LAN `192.168.1.2`, DHCP off, Wi-Fi off, firewall LAN→ACCEPT + WAN MASQUERADE. Luci ships **Homeproxy + sing-box TUN**; paste your nodes and the whole LAN gets ChatGPT / Gemini / Claude / Copilot access with zero client-side proxy apps.

---

## ✨ Features

🔌 **Ready-to-use bypass gateway** – `uci-custom` first-boot script auto-hardcodes LAN IP, disables DHCP/radios, sets firewall rules; no post-flash manual tuning.

🌐 **Homeproxy + sing-box TUN** – Luci visual config, TCP/UDP fully captured, Gemini / Claude / Copilot transparently proxied.

🛡️ **Dual-partition brick recovery** – `luci-app-advanced-reboot` preinstalled; WRT3200ACM dual UBI partitions, one-key switch on bad flash.

💽 **USB storage, all filesystems** – ext4/vfat/ntfs/exfat, plug-and-play U disk.

🎨 **Argon theme + Chinese i18n** – `luci-theme-argon` + full `luci-i18n-*-zh-cn`, friendly web UI.

⚙️ **GitHub Actions auto-build** – after Fork / Use this template, one click yields `sysupgrade.bin` + `factory.img`.

---

## 📟 Hardware & Target

| Item | Detail |
|---|---|
| Device | Linksys WRT3200ACM v1 (Rango) |
| SoC | Marvell Armada 385 88F6820 (Cortex-A9) |
| Arch | mvebu/cortexa9, pkg arch `arm_cortex-a9_vfpv3-d16` |
| Wi-Fi | Marvell 88W8964 (mwlwifi), disabled by default in bypass mode |
| Flash / RAM | 256MB NAND / 512MB DDR3 |
| Image Builder PROFILE | `linksys_wrt3200acm` |

---

## 📁 Repo Layout

BypassRouter-ImmortalWrt-ImageBuild/

├── .github/workflows/image-builder.yml   # Image Builder pipeline

├── packages.list                         # package manifest (Homeproxy / sing-box / kmod-tun …)

├── uci-custom                           # first-boot script → /etc/uci-defaults/99-custom

├── packages/                            # optional extra .ipk (must be arm_cortex-a9_vfpv3-d16)

├── README.md                            # Chinese README

└── README_EN.md                         # This file

---

## 📦 Preinstalled Packages

### Network core (replace defaults)

| Package | Note |
|---|---|
| `dnsmasq-full` | replaces default dnsmasq, DNSSEC / nftables support |
| `wpad-openssl` | replaces wpad-basic, WPA3 / 802.11r |
| `ip-full` | full iproute2 toolset |
| `firewall4` | fw4 / nftables firewall |

### Bypass proxy (Homeproxy TUN)

| Package | Note |
|---|---|
| `luci-app-homeproxy` | Homeproxy Luci frontend (config generation) |
| `luci-i18n-homeproxy-zh-cn` | Homeproxy Chinese translation |
| `sing-box` | proxy engine (TUN device created by sing-box) |
| `kmod-tun` | TUN char device |
| `kmod-nft-tproxy` | TPROXY / redirect dependency |

### Management & theme

| Package | Note |
|---|---|
| `luci-light` + i18n | slim LuCI core |
| `luci-theme-argon` | Argon theme |
| `luci-app-package-manager` | web package manager |
| `luci-app-partexp` | one-click partition expand |
| `luci-app-advanced-reboot` | dual-partition switch / factory flash (brick rescue) |
| `luci-app-autoreboot` | scheduled reboot |
| `autocore` | CPU freq / temp status |

### Storage & USB

| Package | Note |
|---|---|
| `automount` | hotplug auto-mount *(⚠️ 24.10.6 mvebu IB has no `automount` ipk; if build fails, remove this line and use `block-mount`+`blockd`)* |
| `kmod-usb3` / `kmod-usb-storage` / `kmod-usb-storage-uas` | USB 3.0 storage driver |
| `kmod-fs-ext4` / `kmod-fs-vfat` / `kmod-fs-ntfs` / `kmod-fs-exfat` | all-format filesystem support |
| `e2fsprogs` | ext partition tools |

---

## 🔧 Build Steps

### 1️⃣ Create your build repo

Top-right **Use this template → Create a new repository** under your own account.

### 2️⃣ Customize (optional)

| File | Purpose |
|---|---|
| `packages.list` | add/remove packages, one per line; `-pkgname` to drop a default |
| `uci-custom` | first-boot script (change LAN IP / main router IP / root password) |
| `packages/` | extra `.ipk` (arch must be `arm_cortex-a9_vfpv3-d16`) |

### 3️⃣ Trigger build

Go to **Actions → ImmortalWrt Image Builder → Run workflow**, PROFILE defaults to `linksys_wrt3200acm`, wait ~3–8 min.

### 4️⃣ Download firmware

In the Artifacts area download `linksys_wrt3200acm/`, which contains:

| File | Use |
|---|---|
| `*-squashfs-sysupgrade.bin` | upgrade from OpenWrt / ImmortalWrt |
| `*-squashfs-factory.img` | flash from Linksys stock firmware |

---

## 🔌 Flash & Wire

- **Wire**: WRT3200ACM **LAN port only** → main router LAN; WAN port left empty.
- **Dual-partition protection**: hold power button 3s on boot to switch partition, or Luci → System → Advanced Reboot.
- **First-boot automation**: `uci-custom` runs once on first boot – LAN `192.168.1.2`, gateway/DNS `192.168.1.1`, DHCP off, radio0/1 off.

---

## 📱 How clients use the bypass

**Method A · Manual (single device)**

- IP same subnet (e.g. `192.168.1.x`)
- Gateway `192.168.1.2`
- DNS `192.168.1.2`

**Method B · Whole LAN (via main router DHCP)**

- Main router DHCP "default gateway" and "DNS" both set to `192.168.1.2`
- If stock firmware (e.g. Huawei Starlight) only allows editing DNS: set DNS `192.168.1.2` + manual gateway on terminal

✅ Clients need **no proxy app** (Shadowrocket / Clash / v2rayNG etc.); traffic is captured at network layer by TUN.

---

## 🛡️ Homeproxy TUN first run

Luci → Services → HomeProxy:

1. **Node settings**: paste subscription, or manually add VLESS / Trojan / Hysteria2 and pick Main Node.
2. **Client settings**:
   - Proxy mode = **TUN** (TCP + UDP into `singtun0`)
   - Routing: start with **Global** to test, then switch to **Bypass Mainland China**
   - DNS: `8.8.8.8` (international), China DNS leave `223.5.5.5`
3. Enable Homeproxy → status page shows RUNNING, node latency normal.

🔥 **No manual firewall rules**: Homeproxy auto-writes fw4 include, `singtun0` is created by sing-box, bypass MASQUERADE already fixed in `uci-custom`. Luci → Status → Firewall showing `singbox` / `homeproxy` nft chains means injection works.

---

## 🔗 Links

- [ImmortalWrt Official Site](https://www.immortalwrt.org/)
- [ImmortalWrt 24.10.6 mvebu/cortexa9 download](https://downloads.immortalwrt.org/releases/24.10.6/targets/mvebu/cortexa9/)
- [OpenWrt device page (WRT3200ACM)](https://openwrt.org/toh/linksys/linksys_wrt3200acm)

Firmware copyright belongs to the **ImmortalWrt project**. This repo only provides build configuration.

**MIT License · © bjcat119**

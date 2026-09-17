# luci-app-easymesh

Personal fork of EasyMesh for OpenWrt / ImmortalWrt. Batman-adv 802.11s mesh with dual-band backhaul, wired fallback, split-band client SSIDs, and 802.11k/v/r fast roaming.

## Credits

- **Original authors:**
  - dz <dingzhong110@gmail.com> — [torguardvpn/luci-app-easymesh](https://github.com/torguardvpn/luci-app-easymesh)
- **Upstream fork:** [kenzok78/luci-app-easymesh](https://github.com/kenzok78/luci-app-easymesh)
- **Bugfix branch:** [mobing8/luci-app-easymesh-dawn](https://github.com/mobing8/luci-app-easymesh-dawn)
- **Current maintainer:** Arafat Rahman Zami Mondol <zamimondol@gmail.com>

## Features

- Batman-adv 802.11s mesh, dual-band + Wired backhaul with loop prevention (available on both Server and Client)
- Port is automatically removed from the LAN bridge on apply
- Link priority: wired → 5 GHz → 2.4 GHz (`throughput_override`)
- Bonding enabled for simultaneous multi-link traffic
- Auto band detection (frequency-based, works on any radio order)
- Split-band client SSIDs (prevents sticky-2.4 GHz)
- WPS / mode-button pairing via `/etc/rc.wps/` dispatcher
- 60-second rollback watchdog for IP changes
- AP Mode (Dumb AP) for client nodes
- Live latency monitoring for wireless peers (smoothed over 3 pings)
- Live RX/TX bandwidth rates for wireless and wired links
- Neighbor MAC + last-seen for wired backhaul (from batman originator table)
- Hotplug + init + optional cron fallback for link priority
- Variants: openssl / wolfssl / mbedtls

## Prerequisites

- OpenWrt / ImmortalWrt 24.10+
- `kmod-batman-adv`, `batctl-default`, `luci-proto-batman-adv`
- One of: `wpad-mesh-openssl`, `wpad-mesh-wolfssl`, `wpad-mesh-mbedtls` (for WPA3 encryption)

---

## Easy Installation

Run these entirely from `/tmp` — nothing clutters the root filesystem.

**No architecture detection needed.** EasyMesh is `PKG_ARCH=all` — works on every router CPU (x86, MIPS, ARM, AArch64, RISC-V).

---

### Step 0 — Do you need a specific wpad variant?

EasyMesh requires a **mesh-capable wpad** to create the 802.11s backhaul. Most routers already have one installed. Check first:

```sh
# On the router — run this before installing EasyMesh
opkg list-installed 2>/dev/null | grep wpad
# or for OpenWrt 25.12+:
apk list --installed 2>/dev/null | grep wpad
```

**Interpreting the output:**

| What you see | What to do |
|---|---|
| `wpad-mesh-openssl`, `wpad-mesh-wolfssl`, or `wpad-mesh-mbedtls` | ✅ You already have it. **Skip to Step 1.** |
| `wpad-openssl`, `wpad-wolfssl`, `wpad-mbedtls` (no "mesh") | ⚠️ Install a mesh variant — see Step 0b |
| `wpad-basic`, `wpad-basic-mbedtls`, or `wpad-mini` | ❌ Cannot do mesh. Install a mesh variant — see Step 0b |
| Nothing (no wpad) | ❌ Install a mesh variant — see Step 0b |

#### Step 0b — Install a wpad variant (only if needed)

Pick **one** variant. Match what your distribution uses by default:

| Variant | Recommended for |
|---|---|
| `wpad-openssl` | Most OpenWrt builds (default TLS) |
| `wpad-wolfssl` | ImmortalWrt 25.12+ (default TLS) |
| `wpad-mbedtls` | OpenWrt 24.10 default when openssl is unavailable |

**Fastest path — install the meta-package instead.** These install the app **and** the wpad in one shot. If you're unsure, use this:

```sh
# OpenWrt ≤ 24.10 (opkg) — pick ONE
cd /tmp && wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh-wpad-openssl_2.4.1-r1_all.ipk && opkg install luci-app-easymesh-wpad-openssl_2.4.1-r1_all.ipk

# OpenWrt ≥ 25.12 (apk) — pick ONE
cd /tmp && wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh-wpad-openssl-2.4.1-r1.apk && apk add --allow-untrusted luci-app-easymesh-wpad-openssl-2.4.1-r1.apk
```

Swap `openssl` for `wolfssl` or `mbedtls` in both the URL and the package name if you prefer a different variant.

**If you already have wpad and just want to install the wpad separately** (without the meta-package):

```sh
# opkg (≤ 24.10)
opkg install wpad-openssl        # or wpad-wolfssl / wpad-mbedtls
opkg remove wpad-basic-mbedtls   # if you're replacing a basic variant

# apk (≥ 25.12)
apk add wpad-openssl             # or wpad-wolfssl / wpad-mbedtls
apk del wpad-basic-mbedtls       # if you're replacing a basic variant
```

After installing or replacing wpad, reboot once:

```sh
reboot
```

> **Note:** Do not install two mesh-capable wpad variants at the same time — they conflict. If you switch variants, remove the old one first.

---

### Step 1 — Install EasyMesh

**Skip this step if you already installed a `-wpad-*` meta-package above — it includes the main app.**

#### OpenWrt ≤ 24.10 — opkg

```sh
cd /tmp && \
opkg update && \
wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh_2.4.1-r1_all.ipk && \
opkg install luci-app-easymesh_2.4.1-r1_all.ipk && \
rm -f /tmp/luci-indexcache /tmp/luci-modulecache/* && \
/etc/init.d/rpcd restart && /etc/init.d/uhttpd restart
```

#### OpenWrt ≥ 25.12 — apk

```sh
cd /tmp && \
wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh-2.4.1-r1.apk && \
apk add --allow-untrusted luci-app-easymesh-2.4.1-r1.apk && \
rm -f /tmp/luci-indexcache /tmp/luci-modulecache/* && \
/etc/init.d/rpcd restart && /etc/init.d/uhttpd restart
```

---

### ⚡ **Single command (auto-detects opkg vs apk)** ⚡

Run this one-liner. It picks `apk` or `opkg` automatically based on which is present.

```sh
cd /tmp && \
if command -v apk >/dev/null 2>&1; then \
  echo "Detected apk — OpenWrt 25.12+" && \
  wget -O easymesh.pkg https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh-2.4.1-r1.apk && \
  apk add --allow-untrusted easymesh.pkg; \
else \
  echo "Detected opkg — OpenWrt 24.10 or older" && \
  opkg update && \
  wget -O easymesh.pkg https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh_2.4.1-r1_all.ipk && \
  opkg install easymesh.pkg; \
fi && \
rm -f /tmp/luci-indexcache /tmp/luci-modulecache/* && \
/etc/init.d/rpcd restart && /etc/init.d/uhttpd restart
```

---

### Step 2 — Install Chinese (Simplified) translation (optional)

```sh
# opkg (≤ 24.10)
cd /tmp && wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-i18n-easymesh-zh-cn_2.4.1-r1_all.ipk && opkg install luci-i18n-easymesh-zh-cn_2.4.1-r1_all.ipk

# apk (≥ 25.12)
cd /tmp && wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-i18n-easymesh-zh-cn-2.4.1-r1.apk && apk add --allow-untrusted luci-i18n-easymesh-zh-cn-2.4.1-r1.apk
```

---

### Offline / tarball install (no internet on router)

Download on your PC, copy over, extract:

```bash
# On your PC
wget https://github.com/arafatrahmanzami/luci-app-easymesh/releases/download/2.4.1-r1/luci-app-easymesh-2.4.1-r1-full.tar.gz
scp luci-app-easymesh-2.4.1-r1-full.tar.gz root@192.168.1.1:/tmp/
```

```sh
# On the router
cd / && tar xzf /tmp/luci-app-easymesh-2.4.1-r1-full.tar.gz
chmod +x /etc/init.d/easymesh /etc/hotplug.d/iface/30-easymesh-priority /etc/rc.wps/easymesh-pair
rm -f /tmp/luci-indexcache /tmp/luci-modulecache/*
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```

---

### Source code

- [Download `.zip`](https://github.com/arafatrahmanzami/luci-app-easymesh/archive/refs/tags/2.4.1-r1.zip)
- [Download `.tar.gz`](https://github.com/arafatrahmanzami/luci-app-easymesh/archive/refs/tags/2.4.1-r1.tar.gz)

---

### After installation

1. Open LuCI: `http://<router-ip>/cgi-bin/luci/`
2. Go to **Network → EasyMesh**
3. Pick a role (**Server**, **Client**, **Node**) and a preset
4. Click **Save & Apply**
5. If the menu doesn't show, hard-refresh: **Ctrl+Shift+R**

### What gets installed

| Path | Purpose |
|---|---|
| `/usr/lib/lua/luci/controller/easymesh.lua` | LuCI controller |
| `/usr/lib/lua/luci/model/cbi/easymesh.lua` | Configuration UI |
| `/etc/init.d/easymesh` | Init script (bat0, mesh VIFs, wired backhaul, priority) |
| `/etc/config/easymesh` | UCI config |
| `/etc/hotplug.d/iface/30-easymesh-priority` | Reapplies link priority on interface up |
| `/etc/rc.wps/easymesh-pair` | WPS button handler |
| `/etc/uci-defaults/luci-easymesh` | First-run setup |
| `/usr/share/luci/menu.d/luci-app-easymesh.json` | Menu entry |
| `/usr/share/rpcd/acl.d/luci-app-easymesh.json` | ACL permissions |

---

## Quick start

1. **Server** (has internet): EasyMesh → role: **Server**, band mode: **Dual-band**
2. **Client** (extender): role: **Client**, same **Mesh ID**, **AP Mode: ON**, static IP
3. **Both**: same passwords, **K/V/R** on

---

# Changelog

## [2.4.1-r1] - 2026-09-16

### Added

- Wired backhaul dropdown now shown for Server role too (was Client/Node only)
- Port status labels: `(in bridge - will be removed on apply)` / `(free)`
- Filter to only `ethN` / `lanN` — excludes CPU ports and macvlan interfaces
- Link priority via `throughput_override` (wired 1 Gbit, 5 GHz 600 Mbit, 2.4 GHz 30 Mbit)
- Auto band detection via frequency (`iw dev`)
- Bonding enabled in bat0 for simultaneous multi-link traffic
- WPS pairing via `/etc/rc.wps/easymesh-pair` dispatcher (no conflict with wifi-scripts)
- 60-second rollback watchdog for IP changes
- Split-band client SSIDs (2.4 / 5 GHz separate)
- `wait_and_attach_mesh()` — 60s timeout + manual `batctl` attach fallback
- Role-based sanitation: server keeps `wired_if`
- Added UI checkbox for optional cron fallback (default off)
- Live latency monitoring for wireless peers (smoothed over 3 pings)
- Live RX/TX bandwidth rates for wireless and wired links
- Neighbor MAC + last-seen for wired backhaul (from batman originator table)
- Robust priority read (tries multiple batctl syntaxes)
- Hotplug integration for priority reapplication on interface up
- 90-second priority retry loop + second-pass catchup in init script

### Fixed

- Init script wrote escaped quotes to UCI, breaking wireless
- `apply_on_parse=true` in CBI wrote UCI on every page load (lockout)
- Server role could accidentally enable AP mode, disabling DHCP
- Removed `+wpad-mesh` dependency (was causing kconfig recursive dependency)
- `pair()` used `sleep 0.3` — broken on BusyBox
- `pair()` peer check counted blank lines
- `sleep 3; ifup bat0` too short for ath9k mesh
- Count display double-counted wired interfaces

### Changed

- Split into 4 packages: main + 3 wpad metas
- IPK built with `dpkg-deb`

---

## References

- [mobing8/luci-app-easymesh-dawn](https://github.com/mobing8/luci-app-easymesh-dawn)
- [kenzok78/luci-app-easymesh](https://github.com/kenzok78/luci-app-easymesh)
- [torguardvpn/luci-app-easymesh](https://github.com/torguardvpn/luci-app-easymesh)

---

## License

GPL-2.0 (inherited from upstream)

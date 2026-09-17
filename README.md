# luci-app-easymesh

Personal fork of EasyMesh for OpenWrt / ImmortalWrt. Batman-adv 802.11s
mesh with dual-band backhaul, wired fallback, split-band client SSIDs,
and 802.11k/v/r fast roaming.

## Credits

- **Original authors:**
  - dz <dingzhong110@gmail.com> — [torguardvpn/luci-app-easymesh](https://github.com/torguardvpn/luci-app-easymesh)
- **Upstream fork:** [kenzok78/luci-app-easymesh](https://github.com/kenzok78/luci-app-easymesh)
- **Bugfix branch:** [mobing8/luci-app-easymesh-dawn](https://github.com/mobing8/luci-app-easymesh-dawn)
- **Current maintainer:** Arafat Rahman Zami Mondol <zamimondol@gmail.com>


## Features

- Batman-adv 802.11s mesh, dual-band + Wired backhaul with loop prevention (available on both Server and Client)
- Port is automatically removed from the LAN bridge on apply
- Link priority: wired -> 5 GHz -> 2.4 GHz (throughput_override)
- Bonding enabled for simultaneous multi-link traffic
- Auto band detection (frequency-based, works on any radio order)
- Split-band client SSIDs (prevents sticky-2.4 GHz)
- WPS / mode-button pairing via /etc/rc.wps/ dispatcher
- 60-second rollback watchdog for IP changes
- AP Mode (Dumb AP) for client nodes
- Variants: openssl / wolfssl / mbedtls

## Prerequisites

- OpenWrt / ImmortalWrt 24.10+
- kmod-batman-adv, batctl-default, luci-proto-batman-adv
- One of: wpad-mesh-openssl, wpad-mesh-wolfssl, wpad-mesh-mbedtls (for WPA3 encryption)

## Install

### OpenWrt 24.10 and older (opkg)

```sh
# Pick **one** wpad meta package. Example with wolfssl

opkg install luci-app-easymesh_2.4.1-r1_all.ipk
opkg install luci-app-easymesh-wpad-wolfssl_2.4.1-r1_all.ipk
opkg install luci-i18n-easymesh-zh-cn_2.4.1-r1_all.ipk

```
### OpenWrt 25.12+ (apk)

```sh
# Pick **one** wpad meta package. Example with wolfssl

apk add --allow-untrusted luci-app-easymesh-2.4.1-r1.apk
apk add --allow-untrusted luci-app-easymesh-wpad-wolfssl-2.4.1-r1.apk
apk add --allow-untrusted luci-i18n-easymesh-zh-cn-2.4.1-r1.apk

```

### Universal tarball (any OpenWrt)

```sh
scp luci-app-easymesh-2.4.1-r1-full.tar.gz root@router:/tmp/
ssh root@router
cd /
tar xzf /tmp/luci-app-easymesh-2.4.1-r1-full.tar.gz
chmod +x /etc/init.d/easymesh /etc/rc.wps/easymesh-pair
rm -f /tmp/luci-indexcache
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```

## Quick start

1. Server (has internet): EasyMesh -> role: Server, band mode: Dual-band
2. Client (extender): role: Client, same Mesh ID, AP Mode: ON, static IP
3. Both: same passwords, K/V/R on



# Changelog

## [2.4.1-r1] - 2026-09-16

### Added
- Wired backhaul dropdown now shown for Server role too (was Client/Node only)
- Port status labels: "(in bridge - will be removed on apply)" / "(free)"
- Filter to only ethN / lanN — excludes CPU ports and macvlan interfaces
- Link priority via throughput_override (wired 1Gbit, 5GHz 600Mbit, 2.4GHz 30Mbit)
- Auto band detection via frequency (iw dev)
- Bonding enabled in bat0 for simultaneous multi-link traffic
- WPS pairing via /etc/rc.wps/easymesh-pair dispatcher (no conflict with wifi-scripts)
- 60-second rollback watchdog for IP changes
- Split-band client SSIDs (2.4 / 5 GHz separate)
- wait_and_attach_mesh() - 60s timeout + manual batctl attach fallback
- Role-based sanitation: server keeps wired_if
- Added UI checkbox for optional cron fallback (default off)
- Live latency monitoring for wireless peers (smoothed over 3 pings)
- Live RX/TX bandwidth rates for wireless and wired links
- Neighbor MAC + last-seen for wired backhaul (from batman originator table)
- Robust priority read (tries multiple batctl syntaxes)
- Hotplug integration for priority reapplication on interface up
- 90-second priority retry loop + second-pass catchup in init script


### Fixed
- Init script wrote escaped quotes to UCI, breaking wireless
- apply_on_parse=true in CBI wrote UCI on every page load (lockout)
- Server role could accidentally enable AP mode, disabling DHCP
- Removed `+wpad-mesh` dependency (was causing kconfig recursive dependency)
- pair() used sleep 0.3 - broken on BusyBox
- pair() peer check counted blank lines
- sleep 3; ifup bat0 too short for ath9k mesh
- Count display double-counted wired interfaces



### Changed

- Split into 4 packages: main + 3 wpad metas
- IPK built with dpkg-deb







#Old README.md list


https://github.com/mobing8/luci-app-easymesh-dawn
https://github.com/kenzok78/luci-app-easymesh
https://github.com/torguardvpn/luci-app-easymesh












## License

GPL-2.0 (inherited from upstream)

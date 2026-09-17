# Changelog

## [2.4.1-r1] - 2026-09-16

### Added
- Wired backhaul dropdown now shown for Server role too (was Client/Node only)
- Port status labels: "(in bridge - will be removed on apply)" / "(free)"
- Filter to only ethN / lanN — excludes CPU ports and macvlan interfaces
- Link priority via throughput_override (wired 1Gbit, 5GHz 600Mbit, 2.4GHz 30Mbit)
- Auto band detection via frequency (iw dev)
- Bonding enabled in bat0 for simultaneous multi-link traffic
- WPS pairing via /etc/rc.wps/ dispatcher
- 60-second rollback watchdog for IP changes
- Split-band client SSIDs (2.4 / 5 GHz separate)
- wait_and_attach_mesh() - 60s timeout + manual batctl attach fallback
- Role-based sanitation: server keeps wired_if

### Fixed
- Init script wrote escaped quotes to UCI, breaking wireless
- apply_on_parse=true in CBI wrote UCI on every page load (lockout)
- Server role could accidentally enable AP mode, disabling DHCP
- Duplicate +wpad-mesh dependency caused kconfig loop
- pair() used sleep 0.3 - broken on BusyBox
- pair() peer check counted blank lines
- sleep 3; ifup bat0 too short for ath9k mesh
- Count display double-counted wired interfaces

### Changed
- WPS handler moved to /etc/rc.wps/easymesh-pair
- Split into 4 packages: main + 3 wpad metas
- IPK built with dpkg-deb

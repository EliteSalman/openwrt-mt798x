# OpenWrt for IMOU HX21 / LC-HX3001

![OpenWrt logo](include/logo.png)

A customised OpenWrt (`openwrt-25.12` / Linux 6.12) tree for the **IMOU HX21 / LC-HX3001** — the same MediaTek MT7981B-based hardware sold under different regional names. It integrates patches and firmware from `mtk-openwrt-feeds` alongside a practical set of day-to-day networking, diagnostics, and system utilities, keeping the stack completely open-source.

> **Project status: not ready for use.** This tree has **not yet been built with `make`**, let alone flashed or validated on real hardware. Treat everything here as work-in-progress — configs, patches, and package selections are still moving. **Do not flash this on a device you rely on.**

## Target Device

* **IMOU HX21 / LC-HX3001** — MediaTek **MT7981B** (Filogic 820, dual-core Cortex-A53 @ 1.3 GHz, NETSYS v2, 1x PPE, WED + WO MCU) paired with the **MT7976C** dual-band Wi-Fi 6 RF front-end. Both model numbers share identical hardware and use the same target profile in this tree.

## What's Inside

### Wi-Fi & hardware offloading

This build integrates patches, Netfilter HNAT hooks, and firmware blobs from **MediaTek’s `mtk-openwrt-feeds` (`25.12` / `autobuild/unified/filogic`)** on top of the open-source `mt76` and `mtk_eth_soc` drivers. **No proprietary or closed-source kernel drivers are used.**

* **MT7981 Wi-Fi & WED firmware blobs (`.bin`)** — includes the `mtk-openwrt-feeds` (`mac80211/25.12`) firmware set for the MT7981B / MT7976C radio path: **ROM patch (`mt7981_rom_patch.bin`)**, **WM (`mt7981_wm.bin`)**, **WA (`mt7981_wa.bin`)**, and **WED Warp Offload (`mt7981_wo.bin`)** images matching the vendor `mt76` driver patches.
* **Netfilter-based HNAT & WED integration** — adds Netfilter-based **HNAT (Hardware Network Address Translation)** support, bidirectional Wi-Fi-to-Ethernet and Wi-Fi-to-Wi-Fi **WED (Wireless Ethernet Dispatch)** offload paths, and hardware reset/recovery handling on top of the SoC's **NETSYS v2** Frame Engine and **PPE (Packet Processing Engine)**.
* **Bridging, VLAN, and flow-block offload patches** — includes HNAT-adjacent patches for bridge/VLAN traffic offloading and flow-block handling from the `25.12` vendor feed.
* **Vendor VHT and 256-QAM on 2.4 GHz** — enables Broadcom-style vendor VHT extensions (`vendor_vht`) on the 2.4 GHz radio in `mt76` / `mac80211`, allowing supported clients to use VHT rate signalling instead of HT-only MCS and negotiate 256-QAM when supported.

### Hardware crypto acceleration

The **MT7981B** integrates an Inside Secure / Rambus **EIP-97** hardware crypto packet engine (`10320000.crypto`), exposed in Linux 6.12 via the mainline **`crypto-safexcel`** (`kmod-crypto-hw-safexcel`) driver (distinct from the **EIP-197** engine and out-of-tree `crypto-eip` inline IPSec offload driver used on MT7986 and MT7988 SoCs). Kernel crypto engine enablement via `crypto-safexcel` is part of the ongoing bring-up alongside the HNAT and WED paths.

### Day-to-day utilities

A practical set of tools for routing, diagnostics, DNS privacy, and remote management:

* **Networking / routing**
  * OpenVPN
  * WireGuard
  * PPTP
  * 6in4 / 6rd (IPv6 transition)
  * VXLAN
  * Usteer (802.11k/v/r client steering)
  * PBR (Policy Based Routing)
* **DNS privacy**
  * `luci-app-https-dns-proxy` — LuCI Web UI for a lightweight, **RFC 8484-compliant DNS-over-HTTPS (DoH) proxy** service, with `dnsmasq` integration, automatic fallback, and 40+ built-in public DoH resolvers.
* **Time sync**
  * `chrony-nts` — replaces BusyBox `ntpd` with a full **NTP client/server supporting NTS (Network Time Security)** so time synchronisation is cryptographically authenticated.
* **Diagnostics**
  * `mtr`
  * `iputils-ping`
  * `iputils-arping`
  * `tcpdump`
  * `openssl-util`
* **Shell / system**
  * `tmux`
  * `htop`
  * OpenSSH **SFTP** server
  * `tar` with gzip, bzip2, xz, zstd, xattr, and POSIX-ACL support
  * `ubi-utils`
  * `jshn`, `libjson-script`
* **Kernel modules**
  * `kmod-mtd-rw` (writable MTD partitions at runtime)

### Default base-system changes

The default **TLS library** has been switched from **MbedTLS to OpenSSL**:

| From | To |
| :--- | :--- |
| `mbedtls` | `openssl` |
| `apk-mbedtls` | `apk-openssl` |
| `wpad-basic-mbedtls` | `wpad-openssl` |

BusyBox `ntpd` is replaced by `chrony-nts` as noted above.

Additionally:
* **uHTTPd** built with **SSL support** enabled by default.
* **LuCI** web interface included by default.

> **Note:** More changes are coming. This tree is actively developed; package selection, offload configuration, and defaults may change between builds — and it has not been compiled once yet, so expect breakage.

## Development

To build your own firmware you need a GNU/Linux, BSD, or macOS system on a **case-sensitive filesystem**. Cygwin is unsupported due to the lack of a case-sensitive filesystem.

### Related Repositories

* [LuCI Web Interface](https://github.com/openwrt/luci): Modular web interface to manage the device via a browser.
* [OpenWrt Packages](https://github.com/openwrt/packages): Community repository of ported packages.
* [OpenWrt Routing](https://github.com/openwrt/routing): Packages focused on routing protocols and policy routing.
* [OpenWrt Video](https://github.com/openwrt/video): Packages focused on display servers and clients.
* [MediaTek `mtk-openwrt-feeds`](https://git01.mediatek.com/plugins/gitiles/openwrt/feeds/mtk-openwrt-feeds/): Vendor patches, MT7981/WO firmware blobs, Netfilter HNAT support, and offload work for the open-source `mt76` and `mtk_eth_soc` drivers. **No proprietary kernel drivers are used.**

## Support Information

For supported hardware details, see the [OpenWrt Hardware Database](https://openwrt.org/supported_devices).

### Documentation

* [Quick Start Guide](https://openwrt.org/docs/guide-quick-start/start)
* [User Guide](https://openwrt.org/docs/guide-user/start)
* [Developer Documentation](https://openwrt.org/docs/guide-developer/start)
* [Technical Reference](https://openwrt.org/docs/techref/start)

### Support Community

* [Forum](https://forum.openwrt.org): Usage, projects, discussions, and hardware advice.
* [Support Chat](https://webchat.oftc.net/#openwrt): Channel `#openwrt` on **oftc.net**.

## License

OpenWrt is licensed under GPL-2.0. Modifications in this tree are released under the same licence.

---

*This is a community, device-specific fork of OpenWrt. For upstream support and general OpenWrt questions, refer to the official OpenWrt channels linked above.*

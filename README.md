# OpenWrt for IMOU HX21 / LC-HX3001

![OpenWrt logo](include/logo.png)

A customized OpenWrt build for the **IMOU HX21 / LC-HX3001** — the same MediaTek-based hardware sold under different regional names. The goal is **better Wi-Fi and hardware offloading out of the box**, plus a practical set of **day-to-day utilities**, without sacrificing the open-source nature of the stack.

Sunshine!

> ⚠️ **Project status: not ready for use.** This tree has **not yet been built with `make`**, let alone flashed or validated on real hardware. Treat everything here as work-in-progress — configs, patches, and package selections are still moving. **Do not flash this on a device you rely on.**

## Target Device

* **IMOU HX21 / LC-HX3001** — identical hardware, different regional naming. Both are supported by the same target profile in this tree.

## What's Inside

### Better Wi-Fi & hardware offloading out of the box

This build integrates patches and firmware from **MediaTek’s `mtk-openwrt-feeds`** to improve the open-source `mt76` driver and the SoC’s hardware offloading paths. **No proprietary or closed-source drivers are used.** The feed provides:

* **Updated Wi-Fi firmware blobs (`.bin`)** — MediaTek publishes newer firmware images through `mtk-openwrt-feeds` than what is typically shipped in upstream OpenWrt at a given moment. These include DSP, ROM patch, WA/WM, and EEPROM blobs for the MT7996 family, among others. Using the latest blobs can improve stability, fix WED-reset edge cases, and unlock vendor-tuned behaviour.

* **Upstream `mt76` already ships WED and hardware flow offloading — but it lags behind.** Upstream `mt76` has the basic WED support and generic flow-offload hooks, yet it is **missing pieces** that MediaTek's feed carries: newer WED firmware, flow-block refactors, VLAN/bridging offload fixes, and generation-specific tuning. In short, the upstream support exists on paper but is incomplete and behind on the MediaTek-specific paths.

* **Upstream lacks hardware network processing.** Upstream OpenWrt's flow offload is software-centric, and upstream `mt76` does not fully exploit the SoC's **HNAT (Hardware Network Address Translation)** engine or the **WED (Wireless Ethernet Dispatch)** hardware path. `mtk-openwrt-feeds` carries the missing glue: HNAT bindings, sanity checks, Wi-Fi-to-Wi-Fi offload (Warp 2.0 generation onward), and the matching WED firmware that actually makes hardware network processing work in practice.

* **Better hardware offloading for routing, bridging, and HNAT** — beyond pure NAT, the feed includes HNAT-adjacent fixes for bridging, VLAN handling, and flow-block refactoring. These are the kinds of vendor patches that upstream `mt76` / `net-next` may not yet carry for a given MediaTek generation.

* **VHT and 256-QAM on 2.4 GHz** — this tree enables **802.11ac (VHT) and 256-QAM on the 2.4 GHz band**, which upstream `mt76` typically does not enable for MediaTek 2.4 GHz radios. This is a custom addition implemented here to squeeze more throughput out of 2.4 GHz, at the obvious cost of interoperability with clients that do not support it.

In short: upstream `mt76` gives you the open-source driver and a baseline WED/flow-offload path; `mtk-openwrt-feeds` gives you **fresher firmware, HNAT/WED tuning, and vendor offload patches** on top of it — and this tree adds **VHT + 256-QAM on 2.4 GHz** on top of that.

### Crypto EIP & IPSec offloading (work in progress)

MediaTek’s **EIP (Elliptic IP) crypto engine** — specifically the **EIP197** on MT7988-class SoCs — is exposed through a separate `crypto-eip` driver in `mtk-openwrt-feeds`. The feed contains work on:

* **IPSec inline-mode offload** for EIP197, including VLAN support and `NETIF_F_HW_ESP` feature propagation.
* **Debugfs controls** to dynamically enable IPSec hardware offload on a given net device at runtime, rather than requiring a rebuild.
* **IPv6 IPSec and HNAT crypto-offload binding** refactors.

**Status in this tree:** EIP/IPSec offloading is **not fully implemented or validated here yet**. The feed provides the patches, but wiring them into this device’s kernel config, DTS, and userspace (strongSwan/libreswan) is ongoing work. Treat this as a **future/experimental capability**, not a shipped feature.

### Day-to-day utilities

A practical set of tools for routing, diagnostics, and remote management:

* **Networking / routing**
  * OpenVPN
  * WireGuard
  * PPTP
  * 6in4 / 6rd (IPv6 transition)
  * VXLAN
  * Usteer (802.11k/v/r client steering)
  * PBR (Policy Based Routing)
* **Time sync**
  * `chrony-nts` — replaces BusyBox `ntpd` with a full **NTP client using NTS (Network Time Security)**, so time sync is authenticated rather than plaintext NTP.
* **Diagnostics**
  * `mtr`
  * `iputils-ping`
  * `iputils-arping`
  * `tcpdump`
  * `strace` (built without libdw / libunwind)
  * `openssl-util`
* **Shell / system**
  * `tmux`
  * `htop`
  * OpenSSH **SFTP** server
  * `tar` with gzip, bzip2, xz, zstd, xattr and POSIX-ACL support
  * `ubi-utils`
  * `jshn`, `libjson-script`
* **Kernel modules**
  * `kmod-mtd-rw` (writable MTD partitions at runtime)

### Default base-system changes

The default **TLS library** has been switched from **MbedTLS to OpenSSL** to keep the stack consistent and modern:

| From                 | To               |
| -------------------- | ---------------- |
| `mbedtls`            | `openssl`        |
| `apk-mbedtls`        | `apk-openssl`    |
| `wpad-basic-mbedtls` | `wpad-openssl`   |

Confirmed in the shipped `.config`: `CONFIG_PACKAGE_openssl-util=y` while `CONFIG_PACKAGE_mbedtls-util` is **not set**.

BusyBox NTP is replaced by `chrony-nts` as noted above.

Additionally:
* **uHTTPd** built with **SSL support** enabled by default.
* **LuCI** web interface included by default.

> **Note:** More things are coming. This tree is actively developed; package selection, offload configuration, and defaults may change between builds — and it has not been built once yet, so expect breakage.

## Development

To build your own firmware you need a GNU/Linux, BSD or macOS system (case-sensitive filesystem required). Cygwin is unsupported because of the lack of a case-sensitive filesystem.

### Related Repositories

* [LuCI Web Interface](https://github.com/openwrt/luci): Modern and modular interface to control the device via a web browser.
* [OpenWrt Packages](https://github.com/openwrt/packages): Community repository of ported packages.
* [OpenWrt Routing](https://github.com/openwrt/routing): Packages specifically focused on (mesh) routing.
* [OpenWrt Video](https://github.com/openwrt/video): Packages specifically focused on display servers and clients (Xorg and Wayland).
* [MediaTek `mtk-openwrt-feeds`](https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds): Vendor patches, firmware blobs, and offload work for the open-source `mt76` driver. **No proprietary drivers are used.**

## Support Information

For a list of supported devices see the [OpenWrt Hardware Database](https://openwrt.org/supported_devices).

### Documentation

* [Quick Start Guide](https://openwrt.org/docs/guide-quick-start/start)
* [User Guide](https://openwrt.org/docs/guide-user/start)
* [Developer Documentation](https://openwrt.org/docs/guide-developer/start)
* [Technical Reference](https://openwrt.org/docs/techref/start)

### Support Community

* [Forum](https://forum.openwrt.org): For usage, projects, discussions and hardware advice.
* [Support Chat](https://webchat.oftc.net/#openwrt): Channel `#openwrt` on **oftc.net**.

## License

OpenWrt is licensed under GPL-2.0. Modifications in this tree are released under the same license.

---

*This is a community, device-specific fork of OpenWrt. For upstream support and general OpenWrt questions, please refer to the official OpenWrt channels linked above.*

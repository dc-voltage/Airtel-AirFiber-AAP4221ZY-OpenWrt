# Airtel Zyxel AAP4221ZY OpenWrt Device Profile & Backups

- **Device Model:** Airtel AirFiber AAP4221ZY (`airtel,aap4221zy`)
- **Manufacturer:** Zyxel Communications Corp
- **MAC Address:** `44:63:c2:20:00:1a`
- **SoC / Chipset:** MediaTek MT7981B (Filogic 820) Cortex-A53 dual-core (ARM64)
- **RAM:** 512 MB
- **Storage:** NAND Flash with UBI (`/dev/ubi0_3` overlay ~93.4 MB)
- **Wi-Fi:** MediaTek MT7981 Dual-Band (2.4GHz Wi-Fi 6 HE40 & 5GHz VHT80)
- **Current IP:** `192.168.32.1/24` (Bridge: `br-lan`)
- **Default SSH Access:**
  - **Port:** `22` (Dropbear)
  - **User:** `root`
  - **Password:** `Call@8848`
- **Web UI:** `http://192.168.32.1/cgi-bin/luci/`

---

## Directory Contents

- **`backups/`**:
  - `factory.bin` (2.0 MB) — **CRITICAL**: Contains factory RF calibration and MAC address data.
  - `fip.bin` (1.8 MB) — U-Boot / ARM Trusted Firmware partition.
  - `uboot-env.bin` (512 KB) — U-Boot environment variables.
  - `mtd_layout.txt` — Partition map from `/proc/mtd`.
- **`configs/`**:
  - `uci_export.txt` — Complete export of all UCI configuration settings.
  - `etc/config/` — Pristine copy of `/etc/config` (firewall, network, wireless, dhcp, sqm, etc.).
- **`logs/`**:
  - `dmesg.log` — Kernel ring buffer output.
  - `logread.log` — System log read output.
- **Root directory firmware images**:
  - `openwrt-mediatek-filogic-airtel_aap4221zy-initramfs-kernel.bin`
  - `openwrt-mediatek-filogic-airtel_aap4221zy-squashfs-sysupgrade.bin`

---

## Applied Fixes & Status

1. **Firewall Zone Fix for LAN Internet:**
   - Moved `wwan` (`phy1-sta0` wireless client) into the `wan` firewall zone with masquerading enabled.
   - Restored full internet routing for all LAN clients connecting to `192.168.32.1`.
2. **SQM Interface Correction:**
   - Corrected non-existent `eth1` target to actual interface in `sqm` config.
3. **De-branding to Pure OpenWrt / Zyxel:**
   - Model name updated to `Zyxel EX4221-B0 (OpenWrt Pure AX3000)` in LuCI web UI via `/etc/board-base.json` and persistent `/etc/rc.local`.
4. **Hardware Flow Offloading (MediaTek PPE Acceleration):**
   - Enabled `flow_offloading='1'` and `flow_offloading_hw='1'` in `/etc/config/firewall` for gigabit throughput and 0% CPU consumption.
5. **High-Speed WAN Gateway Default:**
   - Disabled slow overseas VPN repeater (`wifinet2` / `phy1-sta0`).
   - Bound default gateway to gigabit Ethernet WAN (`192.168.28.1`) with metric 10. Ping dropped to ~32ms, speed jumped to 260+ Mbps.
6. **Wi-Fi Range & Channel Boost:**
   - 2.4GHz: `Nokia-2.4Gz` (HE40, Channel 1, 30 dBm).
   - 5GHz: `Nokia-5Gz` (HE80, Channel 149, 30.00 dBm / 1000 mW Full Power).
   - Security: WPA2-PSK / WPA3-SAE mixed encryption with key `DC@88488`.
7. **Wi-Fi 6 Advanced Features (BSS Coloring, MU-MIMO, Beamforming):**
   - **BSS Coloring (Spatial Reuse):** Enabled dynamically on both 2.4GHz & 5GHz (`he_bss_color_enabled='1'`, `he_spr_sr_control=3`) avoiding co-channel interference from neighbor routers.
   - **MU-MIMO (Downlink & Uplink):** Explicitly enabled (`mu_beamformer='1'`, `he_mu_beamformer='1'`, full HE MU-EDCA QoS scheduling enabled) allowing parallel multi-device communication.
   - **SU & MU Beamforming:** Explicit sounding active (`su_beamformer='1'`, `he_su_beamformer='1'`, 2x2 antennas).
   - **Implicit Beamforming (`implicit_txbf`):** Activated in `/sys/kernel/debug/ieee80211/phy*/mt76/implicit_txbf` and made persistent in `/etc/rc.local` to beamform even to legacy clients without sounding support.

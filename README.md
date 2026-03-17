# luci-app-airoha-npu

LuCI application for monitoring and managing Airoha AN7581 SoC on OpenWrt. Provides real-time visibility into the NPU, CPU frequency, WiFi offload, Frame Engine, and PPE flow offload status.

## Screenshots

### CPU Frequency & Overclock
![CPU Frequency](screenshots/cpu-frequency.png)

### NPU & Offload Engine
![NPU & Offload Engine](screenshots/npu-offload-engine.png)

### PPE Flow Offload Entries
![PPE Flow Table](screenshots/ppe-flow-table.png)

## Features

### CPU Frequency Management
- Current frequency display with visual bar
- Governor selection (performance, ondemand, schedutil, etc.)
- Max frequency selection from available OPP entries
- Direct PLL overclock (500-1600 MHz) with register programming

### NPU & Offload Engine
- NPU firmware version (TLB format), clock, core count
- NPU load status and reserved memory regions
- WiFi token pool health indicator
- PPE flow offload summary (bound / total entries)
- Per-band WiFi status (2.4 GHz, 5 GHz, 6 GHz) with:
  - Client count and health indicator
  - NPU vs DMA path badge
  - TX retry rate percentage

### Frame Engine (requires `devmem`)
- **PSE Shared Buffer** usage bar (congestion indicator)
- **GDM port cards** with live TX/RX counters and drop counts:
  - GDM1: Internal Switch (1G LAN3/4)
  - GDM2: WAN (USXGMII 10G)
  - GDM4: LAN2 (USXGMII 10G)
- **CDM offload ratio** bars showing HW-forwarded vs CPU-path packets
- **PSE Port Queue Status** grid (P0-P9) with IQ/OQ depths and drop counts

### PPE Flow Offload Table
- First 100 PPE entries with state (BND/UNB), type (IPv4/IPv6/L2B), 5-tuple, and MAC addresses
- Auto-refreshes every 5 seconds

### Theme Support
- Auto-detects dark/light mode by sampling page background luminance
- Works with Glass, Bootstrap, Bootstrap-dark, and other LuCI themes

## Requirements

- OpenWrt with LuCI
- Airoha AN7581 target (`@TARGET_airoha`)
- PPE debugfs enabled (`/sys/kernel/debug/ppe/entries`)
- `devmem` (busybox applet) for Frame Engine register access
- WiFi token_info debugfs for per-band WiFi stats (`/sys/kernel/debug/ieee80211/phy0/mt76/token_info`)

## Installation

### From OpenWrt Build

```bash
# Enable in menuconfig
make menuconfig
# Navigate to: LuCI -> Applications -> luci-app-airoha-npu

# Build
make package/feeds/luci/luci-app-airoha-npu/compile
```

### Manual Installation

```bash
# Copy files to router
scp root/usr/libexec/rpcd/luci.airoha_npu root@router:/usr/libexec/rpcd/
scp root/usr/share/luci/menu.d/luci-app-airoha-npu.json root@router:/usr/share/luci/menu.d/
scp root/usr/share/rpcd/acl.d/luci-app-airoha-npu.json root@router:/usr/share/rpcd/acl.d/
scp htdocs/luci-static/resources/view/airoha_npu/status.js root@router:/www/luci-static/resources/view/airoha_npu/

# Set permissions and restart
ssh root@router 'chmod +x /usr/libexec/rpcd/luci.airoha_npu && /etc/init.d/rpcd restart'
```

## Data Sources

| Data | Source |
|------|--------|
| NPU status | `/sys/bus/platform/drivers/airoha-npu/`, `dmesg` |
| CPU frequency | `/sys/devices/system/cpu/cpufreq/policy0/` |
| Overclock PLL | `devmem` registers (0x1fa202b4, 0x1fa202b8) |
| PPE entries | `/sys/kernel/debug/ppe/{entries,bind}` |
| WiFi token pool | `/sys/kernel/debug/ieee80211/phy0/mt76/token_info` |
| WiFi station stats | `iw dev <iface> station dump` |
| Frame Engine GDM/CDM/PSE | `devmem` registers (0x1fb50xxx-0x1fb53xxx) |

## Version History

### v1.0.0
- CPU frequency management with governor and overclock controls
- NPU & Offload Engine monitoring with WiFi band cards
- Frame Engine visualization (GDM ports, CDM offload ratio, PSE queues)
- PPE flow offload table
- Theme-adaptive dark/light mode detection

## License

Apache-2.0

## Author

Ryan Chen - Created for W1700K router (Airoha AN7581 + MT7996 BE19000)

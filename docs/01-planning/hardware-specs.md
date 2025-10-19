# Hardware Specifications

## Firewall

### Glovary i3 N355 6 LAN Mini PC

**Processor**
- Model: Intel N355 (Alder Lake-N)
- Cores/Threads: 8C/8T
- Base Clock: 3.9 GHz
- Architecture: Intel 7 (10nm)
- TDP: 15W

**Memory**
- Installed: 8GB DDR5 SO-DIMM
- Speed: 4.8 GT/s
- Max Capacity: 32GB
- **Upgrade Planned**: 16GB DDR5

**Storage**
- Type: 512GB SSD (PCIe x1)
- Interface: Solid State

**Network Interfaces**
- Total Ports: 6x 2.5 Gigabit Ethernet
  - Port 1: WAN (to ISP router)
  - Port 2: LAN Trunk (to managed switch)
  - Ports 3-6: Available for expansion
- Wireless: Wi-Fi (disabled for security)

**Graphics**
- Intel UHD Graphics (integrated)
- Video Output: 2x HDMI, 1x USB-C with DisplayPort

**Additional Ports**
- USB: 6 total ports
- Power: Type-A 2-pin (North American)

**Dimensions**
- Size: 7.09" x 4.92" x 2.17"
- Weight: 1.2 kg (2.6 lbs)

**Operating System**
- Pre-installed: RouterOS
- Current: pfSense CE 2.7.x

**Performance Expectations**
- Routing: 2.5 Gbps (line rate)
- Firewall + NAT: 2.0 Gbps
- IDS/IPS (Suricata): 1.5-2.0 Gbps
- WireGuard VPN: 1.0-1.5 Gbps

**Purchase Info**
- Vendor: [Amazon/Other]
- Price: $[XXX]
- Date: [Purchase Date]

---

## Proxmox Nodes

### Node 1: pve (Primary)

**Hostname**: pve  
**Purpose**: Primary compute node for VMs and containers

**Processor**
- Model: AMD Ryzen 9 5950X
- Cores/Threads: 16C/32T
- Base/Boost: 3.4 / 4.9 GHz
- Architecture: Zen 3
- TDP: 105W

**Memory**
- Capacity: 128GB DDR4
- Speed: [Speed if known]
- Configuration: [Channel/DIMM config]

**Storage**
- Primary: 2TB NVMe SSD (OS, VMs)
- Secondary: 14TB HDD (bulk storage, media)
- Total: 16TB

**Network**
- NIC: 2.5 Gigabit Ethernet
- Connection: Mokerlink Switch Port 2 (VLAN 20)

**Current IP**: 192.168.78.128  
**New IP**: 10.0.20.10

**Proxmox Version**: [Version]

**Planned Workloads**
- Docker host (Services VLAN)
- Media server VMs (Plex/Jellyfin)
- Development VMs
- Database servers

---

### Node 2: amdpve (Secondary)

**Hostname**: amdpve  
**Purpose**: Secondary compute node

**Processor**
- Model: AMD Ryzen 7 5700G
- Cores/Threads: 8C/16T
- Base/Boost: 3.8 / 4.6 GHz
- Architecture: Zen 3
- Integrated GPU: Radeon Graphics
- TDP: 65W

**Memory**
- Capacity: 64GB DDR4

**Storage**
- Primary: 1TB NVMe SSD

**Network**
- NIC: 2.5 Gigabit Ethernet
- Connection: Mokerlink Switch Port 3 (VLAN 20)

**Current IP**: 192.168.0.128  
**New IP**: 10.0.20.11

**Proxmox Version**: [Version]

**Planned Workloads**
- Testing/staging VMs
- Security tools (Red Team)
- Monitoring stack (Purple Team)
- Backup target

---

## Network Infrastructure

### Managed Switch

**Model**: Mokerlink G30800B-C  
**Type**: Managed 2.5G Switch

**Specifications**
- Ports: 8x 2.5 Gigabit Ethernet
- VLAN Support: 802.1Q, Port-based
- Management: Web UI
- Backplane: 40 Gbps

**Current IP**: [IP Address]  
**New IP**: 10.0.20.2

**Port Assignments**
| Port | VLAN | Purpose | Device |
|------|------|---------|--------|
| 1 | Trunk | All VLANs | pfSense Port 2 |
| 2 | 20 | Management | Proxmox Node 1 (pve) |
| 3 | 20 | Management | Proxmox Node 2 (amdpve) |
| 4 | 60 | Gaming | Gaming PC |
| 5 | 50 | IoT | NICGIGA Switch (uplink) |
| 6 | 62 | Purple Team | Raspberry Pi 5 |
| 7 | 61 | Red Team | Raspberry Pi 4 |
| 8 | - | Available | Future expansion |

---

### Unmanaged Switch

**Model**: NICGIGA 5-Port 2.5G  
**Type**: Unmanaged Switch

**Specifications**
- Ports: 5x 2.5 Gigabit Ethernet
- VLAN: None (unmanaged)
- Purpose: IoT device aggregation

**Connection**: Mokerlink Port 5 (VLAN 50)

**Connected Devices** (all on VLAN 50 - IoT)
- Port 1: Uplink to Mokerlink
- Port 2: Amazon Fire Stick
- Port 3: Smart home hub
- Port 4-5: Additional IoT devices

---

## Client Devices

### Gaming PC

**Hostname**: [Hostname]

**Processor**
- Model: AMD Ryzen 9 9950X3D
- Cores/Threads: 16C/32T

**Memory**
- Capacity: 96GB DDR5

**Graphics**
- GPU: NVIDIA RTX 5090

**Network**
- NIC: 5 Gigabit Ethernet (negotiates to 2.5G)
- Connection: Mokerlink Port 4 (VLAN 60)

**Current IP**: Various  
**New IP**: 10.0.60.50 (static or DHCP reservation)

---

### Raspberry Pi 5 (Purple Team)

**Purpose**: Monitoring, IDS analysis, logging

**Specifications**
- RAM: 8GB
- Storage: NVMe SSD via hat
- Network: Gigabit Ethernet (negotiates to 1G on 2.5G switch)
- Connection: Mokerlink Port 6 (VLAN 62)

**Planned Services**
- Grafana dashboard
- Prometheus metrics
- Log aggregation
- Traffic analysis

**IP**: 10.0.62.10

---

### Raspberry Pi 4 (Red Team)

**Purpose**: Security testing, penetration testing

**Specifications**
- RAM: 8GB
- Power: PoE hat
- Network: Gigabit Ethernet
- Connection: Mokerlink Port 7 (VLAN 61)

**Planned Services**
- Kali Linux tools
- Network scanning
- Vulnerability assessment
- Security training

**IP**: 10.0.61.10

---

### Zimaboard (Future)

**Purpose**: TBD (potential 3rd Proxmox node or dedicated service)

**Status**: Available for deployment

---

## ISP Equipment

### ISP Router/Modem

**Model**: [ISP Router Model]  
**Connection**: 2 Gbps fiber

**Configuration**
- Mode: DMZ to pfSense WAN
- DMZ IP: 192.168.1.10 (pfSense WAN)
- LAN Network: 192.168.1.0/24 (household devices)
- WiFi: [Enabled/Disabled for household]

**Household Devices** (untouched, on 192.168.1.x)
- TVs, phones, tablets, etc.
- Separate from homelab network

---

## Total Cost Breakdown

| Item | Cost | Notes |
|------|------|-------|
| pfSense Firewall (N355) | $XXX | |
| Mokerlink Switch | $XXX | |
| NICGIGA Switch | $XXX | |
| Proxmox Node 1 | $XXX | or existing |
| Proxmox Node 2 | $XXX | or existing |
| Raspberry Pi 5 | $XXX | |
| Raspberry Pi 4 | $XXX | or existing |
| Cables (Cat6/6a) | $XX | |
| **Total** | **$XXXX** | |

---

## Future Hardware Plans

### Potential Upgrades
- [ ] pfSense RAM: 8GB → 16GB DDR5 (~$50-80)
- [ ] 10G network card for Proxmox nodes
- [ ] NAS for centralized storage
- [ ] Dedicated WiFi access points (UniFi/TP-Link Omada)
- [ ] UPS for critical equipment
- [ ] Rack mount solution

### Expansion Ideas
- [ ] 3rd Proxmox node (Zimaboard or new hardware)
- [ ] Dedicated backup server
- [ ] GPU passthrough for AI/ML workloads
- [ ] Dedicated monitoring appliance

---

**Last Updated**: [10/19/2025]
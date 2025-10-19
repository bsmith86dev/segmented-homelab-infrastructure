# segmented-homelab-infrastructure
# Enterprise-Grade Homelab Network

> A fully segmented, VLAN-based homelab network infrastructure with enterprise security features.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![pfSense](https://img.shields.io/badge/pfSense-2.7-blue)](https://www.pfsense.org/)
[![Proxmox](https://img.shields.io/badge/Proxmox-8.x-orange)](https://www.proxmox.com/)

## 🎯 Project Goals

- Migrate from flat network to segmented VLAN architecture
- Implement enterprise-grade security (IDS/IPS, network isolation)
- Enable self-hosting infrastructure for homelab services
- Achieve 2 Gbps internet throughput with full security stack
- Document the entire process for community learning

## 📊 Network Overview

### Hardware
- **Firewall**: Glovary i3 N355 Mini PC (8C/8T @ 3.9GHz, 6x 2.5G ports)
- **Switch**: Mokerlink G30800B-C (8x 2.5G managed switch)
- **Compute**: 
  - Proxmox Node 1: AMD Ryzen 9 5950X, 128GB RAM, 2TB NVMe
  - Proxmox Node 2: AMD Ryzen 7 5700G, 64GB RAM, 1TB NVMe
- **ISP**: 2 Gbps fiber connection

### Network Architecture
```
Internet (2 Gbps)
    ↓
ISP Router (DMZ)
    ↓
pfSense Firewall (N355)
    ├─ VLAN 20: Management (10.0.20.0/24)
    ├─ VLAN 30: Services (10.0.30.0/24)
    ├─ VLAN 40: Media (10.0.40.0/24)
    ├─ VLAN 50: IoT (10.0.50.0/24)
    ├─ VLAN 60: Gaming/Clients (10.0.60.0/24)
    ├─ VLAN 61: Red Team (10.0.61.0/24)
    ├─ VLAN 62: Purple Team (10.0.62.0/24)
    └─ VLAN 70: Guest (10.0.70.0/24)
```

## 🔒 Security Features

- ✅ Network segmentation (8 isolated VLANs)
- ✅ Suricata IDS/IPS (inline threat blocking)
- ✅ pfBlockerNG (DNS/IP filtering)
- ✅ WireGuard VPN (remote access)
- ✅ IoT isolation (no local network access)
- ✅ DMZ from household network

## 📚 Documentation

### Implementation Phases
1. [Planning & Design](docs/01-planning/)
2. [ISP & DMZ Setup](docs/02-implementation/phase1-isp-setup.md)
3. [pfSense Configuration](docs/02-implementation/phase2-pfsense-basic.md)
4. [VLAN Implementation](docs/02-implementation/phase3-vlans.md)
5. [Firewall Rules](docs/02-implementation/phase5-firewall-rules.md)
6. [Proxmox Migration](docs/02-implementation/phase8-proxmox.md)
7. [Security Stack](docs/02-implementation/phase9-security-stack.md)

### Diagrams
- [Network Topology](diagrams/network-topology.png)
- [VLAN Layout](diagrams/vlan-layout.png)

## 🚀 Performance Metrics

| Metric | Target | Achieved |
|--------|--------|----------|
| Internet Throughput | 2 Gbps | TBD |
| IDS/IPS Throughput | 1.5 Gbps | TBD |
| VPN Throughput | 1 Gbps | TBD |
| Firewall Latency | <1 ms | TBD |

## 🛠️ Tech Stack

- **Firewall**: pfSense 2.7.x
- **Virtualization**: Proxmox VE 8.x
- **IDS/IPS**: Suricata
- **DNS Filtering**: pfBlockerNG
- **VPN**: WireGuard
- **Monitoring**: Grafana + Prometheus (planned)
- **Containers**: Docker on LXC

## 📖 Lessons Learned

_Will be updated as project progresses_

## 🙏 Acknowledgments

- Claude (Anthropic) for architecture guidance
- r/homelab community
- pfSense documentation team

## 📝 License

MIT License - See [LICENSE](LICENSE) file

## 📧 Contact

[github.com/bsmith86dev] 

---

**Status**: 🚧 In Progress | **Started**: [Date] | **Last Updated**: [Date]

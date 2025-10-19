# 🏠 Enterprise Homelab Network Infrastructure

> Building a production-grade, VLAN-segmented network from scratch with complete documentation.

[![Project Status](https://img.shields.io/badge/Status-In%20Progress-yellow)](https://github.com/yourusername/homelab-network)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/Docs-Complete-green)](docs/)
[![pfSense](https://img.shields.io/badge/pfSense-2.7.x-blue)](https://www.pfsense.org/)
[![Proxmox](https://img.shields.io/badge/Proxmox-8.x-orange)](https://www.proxmox.com/)

---

## 📋 Project Overview

This project documents the complete migration from a flat home network to an enterprise-grade, segmented infrastructure using VLANs, pfSense firewall, and Proxmox virtualization.

### Goals
- ✅ Implement network segmentation for security
- ✅ Achieve 2 Gbps internet throughput with full security stack
- ✅ Enable self-hosting infrastructure
- ✅ Document every step for community learning
- ✅ Build portfolio-ready technical documentation

---

## 🏗️ Project Status

**Started**: [Start Date]  
**Current Phase**: Phase 0 - Documentation Setup  
**Overall Completion**: 0%

### Implementation Progress

| Phase | Status | Duration | Completion Date |
|-------|--------|----------|-----------------|
| **Phase 0**: Documentation Setup | 🔄 In Progress | 2 hours | - |
| **Phase 1**: ISP Router DMZ | ⬜ Not Started | 30 min | - |
| **Phase 2**: pfSense Basic Setup | ⬜ Not Started | 45 min | - |
| **Phase 3**: VLAN Configuration | ⬜ Not Started | 1 hour | - |
| **Phase 4**: DHCP Services | ⬜ Not Started | 30 min | - |
| **Phase 5**: Firewall Rules | ⬜ Not Started | 2 hours | - |
| **Phase 6**: NAT Configuration | ⬜ Not Started | 30 min | - |
| **Phase 7**: Managed Switch Setup | ⬜ Not Started | 1 hour | - |
| **Phase 8**: Proxmox Migration | ⬜ Not Started | 2 hours | - |
| **Phase 9**: Security Stack | ⬜ Not Started | 3 hours | - |

**Legend**: ⬜ Not Started | 🔄 In Progress | ✅ Complete | ❌ Blocked

---

## 🔧 Hardware Specifications

### Firewall
- **Model**: Glovary i3 N355 6 LAN Mini PC
- **CPU**: Intel N355 (8C/8T @ 3.9 GHz)
- **RAM**: 8GB DDR5 (upgradable to 32GB)
- **Network**: 6x 2.5 Gigabit Ethernet
- **Performance**: 2 Gbps routing, 1.5 Gbps with IDS/IPS

### Compute Nodes

**Primary Node (pve)**
- AMD Ryzen 9 5950X (16C/32T)
- 128GB DDR4 RAM
- 2TB NVMe + 14TB HDD

**Secondary Node (amdpve)**
- AMD Ryzen 7 5700G (8C/16T)
- 64GB DDR4 RAM
- 1TB NVMe

### Networking
- **Managed Switch**: Mokerlink G30800B-C (8x 2.5G)
- **Unmanaged Switch**: NICGIGA (5x 2.5G)
- **ISP**: 2 Gbps fiber connection

### Additional
- Raspberry Pi 5 8GB (Monitoring)
- Raspberry Pi 4 8GB (Security Testing)
- Gaming PC: Ryzen 9 9950X3D, RTX 5090, 5G NIC

[Full hardware specs →](docs/01-planning/hardware-specs.md)

---

## 🌐 Network Architecture

### Topology Overview
```
Internet (2 Gbps)
    ↓
ISP Router (DMZ) ─── Household Network (isolated)
    ↓
pfSense Firewall (Intel N355)
    ├─ VLAN 20: Management (10.0.20.0/24)
    ├─ VLAN 30: Services (10.0.30.0/24)
    ├─ VLAN 40: Media (10.0.40.0/24)
    ├─ VLAN 50: IoT - ISOLATED (10.0.50.0/24)
    ├─ VLAN 60: Gaming/Clients (10.0.60.0/24)
    ├─ VLAN 61: Red Team (10.0.61.0/24)
    ├─ VLAN 62: Purple Team (10.0.62.0/24)
    └─ VLAN 70: Guest (10.0.70.0/24)
    ↓
Mokerlink Managed Switch
    ├─ Proxmox Nodes (Management VLAN)
    ├─ Gaming PC (Client VLAN)
    ├─ IoT Devices (IoT VLAN - isolated)
    └─ Security Lab (Red/Purple Team VLANs)
```

[Detailed network design →](docs/01-planning/network-design.md)

---

## 🔒 Security Features

- ✅ **Network Segmentation**: 8 isolated VLANs by function and trust level
- ✅ **IDS/IPS**: Suricata inline threat blocking (1.5-2 Gbps throughput)
- ✅ **DNS/IP Filtering**: pfBlockerNG for ad/tracker blocking
- ✅ **VPN**: WireGuard for secure remote access (1-1.5 Gbps)
- ✅ **IoT Isolation**: Zero trust - internet only, no LAN access
- ✅ **DMZ**: Homelab separated from household network
- ✅ **Monitoring**: Purple Team VLAN with IDS analysis

[Security architecture →](docs/01-planning/network-design.md#security-zones)

---

## 📚 Documentation

### Planning Documents
- [Hardware Specifications](docs/01-planning/hardware-specs.md)
- [Network Design & Architecture](docs/01-planning/network-design.md)
- [VLAN Scheme & Assignments](docs/01-planning/vlan-scheme.md)

### Implementation Guides
- [Phase 1: ISP Router DMZ Setup](docs/02-implementation/phase1-isp-setup.md)
- [Phase 2: pfSense Basic Configuration](docs/02-implementation/phase2-pfsense-basic.md) *(coming soon)*
- [Phase 3: VLAN Implementation](docs/02-implementation/phase3-vlans.md) *(coming soon)*
- More phases in progress...

### Configuration Files
- [pfSense Configs](docs/03-configuration/pfsense-config-backup/)
- [Switch Configs](docs/03-configuration/switch-config/)
- [Proxmox Network](docs/03-configuration/proxmox-network/)

### Troubleshooting
- [Common Issues & Solutions](docs/04-troubleshooting/common-issues.md)
- [Debug Commands](docs/04-troubleshooting/debug-commands.md)

---

## 📊 Performance Targets

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Internet Throughput (no IDS) | 2.0 Gbps | TBD | ⬜ |
| Internet Throughput (with IDS/IPS) | 1.5 Gbps | TBD | ⬜ |
| Inter-VLAN Latency | <1 ms | TBD | ⬜ |
| WAN Latency | <2 ms | TBD | ⬜ |
| VPN Throughput | 1.0 Gbps | TBD | ⬜ |
| Firewall CPU (2 Gbps load) | <60% | TBD | ⬜ |

---

## 🛠️ Tech Stack

### Infrastructure
- **Firewall**: pfSense CE 2.7.x
- **Virtualization**: Proxmox VE 8.x
- **Networking**: 802.1Q VLANs, 2.5G Ethernet

### Security
- **IDS/IPS**: Suricata
- **DNS Filtering**: pfBlockerNG
- **VPN**: WireGuard

### Monitoring (Planned)
- **Metrics**: Prometheus + Grafana
- **Logs**: ELK Stack or Loki
- **Alerting**: Alertmanager

### Services (Planned)
- **Containers**: Docker on Proxmox LXC
- **Reverse Proxy**: Traefik or Nginx Proxy Manager
- **DNS**: Pi-hole or AdGuard Home
- **Media**: Plex or Jellyfin

---

## 🎯 Key Design Decisions

### Why DMZ Instead of Bridge Mode?
- Maintains household network functionality
- Easier troubleshooting (can bypass pfSense)
- No impact on non-homelab devices

### Why VLAN Trunk vs Direct Ports?
- Scalable - add VLANs without rewiring
- Efficient use of limited pfSense ports
- Industry standard approach

### Why IoT Complete Isolation?
- Zero trust security model
- Prevents IoT devices from accessing sensitive data
- Mitigates smart device vulnerabilities

[Full design rationale →](docs/01-planning/network-design.md#design-decisions--rationale)

---

## 📸 Visual Documentation

### Network Diagrams
- [Logical Topology](diagrams/network-topology.png) *(coming soon)*
- [Physical Layout](diagrams/physical-layout.png) *(coming soon)*
- [VLAN Scheme](diagrams/vlan-layout.png) *(coming soon)*

### Screenshots
- [Implementation Progress](screenshots/)
- [pfSense Configuration](screenshots/pfsense/)
- [Performance Metrics](screenshots/performance/)

---

## 📖 Lessons Learned

*Will be updated as the project progresses*

### Challenges Faced
- TBD

### Solutions Implemented
- TBD

### What I'd Do Differently
- TBD

### Recommendations for Others
- TBD

---

## 🚀 Future Enhancements

### Phase 2 (Post-Deployment)
- [ ] WiFi access points with VLAN support
- [ ] Quality of Service (QoS) traffic shaping
- [ ] Proxmox HA clustering
- [ ] Automated configuration backups

### Phase 3 (Advanced)
- [ ] 10G networking for storage traffic
- [ ] Multi-WAN failover (4G/5G backup)
- [ ] Certificate authority for internal SSL
- [ ] Kubernetes cluster on dedicated VLAN

### Phase 4 (Long-term)
- [ ] Site-to-site VPN (future locations)
- [ ] Dedicated NAS with ZFS
- [ ] GPU passthrough for AI/ML
- [ ] Network access control (802.1X)

---

## 🤝 Contributing

This is a personal project, but I welcome:
- Suggestions for improvements
- Questions about implementation
- Bug reports in documentation
- Security recommendations

Feel free to open an issue or submit a pull request!

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

You are free to:
- Use this documentation for your own homelab
- Modify and adapt to your needs
- Share with others

Attribution appreciated but not required.

---

## 🙏 Acknowledgments

- **Claude (Anthropic)** - Architecture design and documentation assistance
- **r/homelab community** - Inspiration and best practices
- **pfSense team** - Excellent firewall platform
- **Proxmox team** - Outstanding virtualization platform

### Resources Used
- [pfSense Documentation](https://docs.netgate.com/pfsense/)
- [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)
- [Suricata Documentation](https://suricata.readthedocs.io/)
- [r/homelab Wiki](https://www.reddit.com/r/homelab/wiki/index)

---

## 📧 Contact & Portfolio

- **GitHub**: (https://github.com/bsmith86dev)
- **LinkedIn**: (https://www.linkedin.com/in/bsmith86cs)
- **Portfolio**: (https://brandonsaws.com)

---

## 📅 Project Timeline
```
Week 1: Planning & Documentation ────────────► 🔄 In Progress
Week 2: Core Infrastructure Setup ───────────► ⬜ Pending
Week 3: Security Stack Deployment ───────────► ⬜ Pending
Week 4: Service Migration & Testing ─────────► ⬜ Pending
Week 5: Monitoring & Optimization ───────────► ⬜ Pending
Week 6+: Advanced Features ──────────────────► ⬜ Pending
```

---

## 🌟 Project Highlights

**Why This Project Stands Out:**
- 📝 Complete documentation from planning to deployment
- 🔒 Enterprise-grade security on homelab budget
- 📊 Performance benchmarks and real-world metrics
- 🎓 Educational value for network/security enthusiasts
- 💼 Portfolio-quality technical writing

---

**⭐ Star this repo if you find it useful!**

**Last Updated**: 10/19/2025  
**Current Phase**: Documentation Setup  
**Next Milestone**: Phase 1 - ISP DMZ Configuration

---
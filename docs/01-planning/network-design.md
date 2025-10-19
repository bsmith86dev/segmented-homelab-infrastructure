# Network Architecture Design

## Overview

This document outlines the network architecture for a enterprise-grade homelab infrastructure, designed to provide:
- Network segmentation for security and organization
- High-performance routing (2 Gbps throughput)
- Intrusion detection and prevention
- Isolated IoT network
- Separated household and homelab networks

---

## Design Principles

### 1. **Security First**
- Default deny firewall rules
- Network segmentation by function and trust level
- IoT devices completely isolated
- IDS/IPS on all internet traffic

### 2. **Performance**
- 2.5G networking throughout
- Minimal hops between critical services
- Hardware offloading where possible

### 3. **Scalability**
- VLAN-based design allows easy expansion
- Trunk port design supports adding VLANs without hardware changes
- Proxmox clustering ready

### 4. **Separation of Concerns**
- Household network isolated from homelab
- Management network restricted
- Services segmented by function

---

## Network Topology

### Logical Topology
```
┌─────────────────────────────────────────────────────────┐
│                    Internet (2 Gbps)                    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────┐
│              ISP Router (192.168.1.1)                   │
│                                                          │
│  Household Network: 192.168.1.0/24                      │
│  └─ TVs, phones, tablets (untouched)                    │
│                                                          │
│  DMZ to Homelab: 192.168.1.10                           │
└────────────────────────┬────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────┐
│           pfSense Firewall (Intel N355)                 │
│                                                          │
│  WAN: 192.168.1.10/24 (from ISP router)                │
│  LAN Trunk: All VLANs (10.0.0.0/8)                     │
│                                                          │
│  Security Stack:                                         │
│   - Suricata IDS/IPS                                    │
│   - pfBlockerNG (DNS + IP filtering)                    │
│   - WireGuard VPN                                       │
│   - HAProxy (load balancing)                            │
└────────────────────────┬────────────────────────────────┘
                         │
                         ↓ (VLAN Trunk - 2.5G)
┌─────────────────────────────────────────────────────────┐
│        Mokerlink Managed Switch (8x 2.5G)               │
│                                                          │
│  Port 1: Trunk (all VLANs tagged)                       │
│  Port 2: VLAN 20 (pve - Proxmox Node 1)                │
│  Port 3: VLAN 20 (amdpve - Proxmox Node 2)             │
│  Port 4: VLAN 60 (Gaming PC)                            │
│  Port 5: VLAN 50 (NICGIGA uplink - IoT devices)        │
│  Port 6: VLAN 62 (RPi 5 - Purple Team)                 │
│  Port 7: VLAN 61 (RPi 4 - Red Team)                    │
│  Port 8: Available                                       │
└─────────────────────────────────────────────────────────┘
```

### Physical Topology
```
[ISP ONT] ─── [ISP Router] ─┬─ [Household WiFi/Devices]
                             │
                             └─ [Port 4 - DMZ]
                                     ↓
                            [pfSense Port 1 - WAN]
                                     │
                            [pfSense Port 2 - LAN Trunk]
                                     ↓
                        [Mokerlink Switch Port 1 - Trunk]
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
             [Port 2: pve]    [Port 3: amdpve]  [Port 4: Gaming PC]
                 (VLAN 20)        (VLAN 20)        (VLAN 60)
                    │                │                
             [Port 5: NICGIGA]  [Port 6: RPi5]  [Port 7: RPi4]
                 (VLAN 50)        (VLAN 62)        (VLAN 61)
                    │
            [IoT Devices via unmanaged switch]
```

---

## IP Addressing Scheme

### Network Ranges

| Network | VLAN | Subnet | Gateway | DHCP Range | Purpose |
|---------|------|--------|---------|------------|---------|
| **Household** | - | 192.168.1.0/24 | 192.168.1.1 | ISP Router DHCP | Household devices (isolated) |
| **Management** | 20 | 10.0.20.0/24 | 10.0.20.1 | .100-.250 | Proxmox hosts, switches, pfSense |
| **Services** | 30 | 10.0.30.0/24 | 10.0.30.1 | .100-.250 | Docker, VMs, self-hosted apps |
| **Media** | 40 | 10.0.40.0/24 | 10.0.40.1 | .100-.250 | Plex, Jellyfin, media storage |
| **IoT** | 50 | 10.0.50.0/24 | 10.0.50.1 | .100-.250 | Smart devices, cameras, Fire Stick |
| **Gaming/Clients** | 60 | 10.0.60.0/24 | 10.0.60.1 | .100-.250 | Gaming PC, workstations, phones |
| **Red Team** | 61 | 10.0.61.0/24 | 10.0.61.1 | .100-.250 | Security testing, pentesting VMs |
| **Purple Team** | 62 | 10.0.62.0/24 | 10.0.62.1 | .100-.250 | Monitoring, IDS, logging |
| **Guest** | 70 | 10.0.70.0/24 | 10.0.70.1 | .100-.250 | Guest devices (optional) |
| **VPN** | 99 | 10.0.99.0/24 | 10.0.99.1 | .10-.50 | WireGuard VPN clients |

### Static IP Reservations

#### Infrastructure (VLAN 20 - Management)
| Device | IP | MAC Address | Notes |
|--------|-----|-------------|-------|
| pfSense MGT | 10.0.20.1 | - | Gateway |
| Mokerlink Switch | 10.0.20.2 | [MAC] | Management |
| Proxmox Node 1 (pve) | 10.0.20.10 | [MAC] | Primary compute |
| Proxmox Node 2 (amdpve) | 10.0.20.11 | [MAC] | Secondary compute |
| Zimaboard (future) | 10.0.20.12 | [MAC] | 3rd node or services |

#### Client Devices
| Device | VLAN | IP | MAC Address | Notes |
|--------|------|-----|-------------|-------|
| Gaming PC | 60 | 10.0.60.50 | [MAC] | Static or DHCP reservation |
| RPi 5 (Purple) | 62 | 10.0.62.10 | [MAC] | Monitoring |
| RPi 4 (Red) | 61 | 10.0.61.10 | [MAC] | Security testing |

---

## Traffic Flow Patterns

### Internet Access (Allowed)
```
Client Device → VLAN Gateway → pfSense → NAT → ISP Router → Internet
```

### Inter-VLAN Access (Controlled by Firewall)

**Allowed Paths:**
- Management (20) → All VLANs (full access)
- Services (30) → Media (40) - for media management
- Services (30) → Internet
- Gaming (60) → Media (40) - for streaming
- Gaming (60) → Internet
- Purple Team (62) → All VLANs (monitoring)

**Blocked Paths:**
- IoT (50) → Any VLAN (internet only)
- Red Team (61) → All VLANs by default (controlled access)
- Guest (70) → Any VLAN (internet only)

### VPN Access 
**Remote Client → Internet → pfSense WAN → WireGuard → VPN VLAN (99) → Homelab VLANs**
---

## Security Zones

### Zone Classification

| Zone | Trust Level | Purpose | Example VLANs |
|------|-------------|---------|---------------|
| **Trusted** | High | Critical infrastructure | Management (20) |
| **Internal** | Medium-High | Services for personal use | Services (30), Media (40) |
| **Client** | Medium | End-user devices | Gaming (60) |
| **Restricted** | Low | Untrusted devices | IoT (50), Guest (70) |
| **Isolated** | Minimal | Security testing | Red Team (61) |
| **Monitoring** | High | Security monitoring | Purple Team (62) |
| **Household** | Isolated | Separate network | 192.168.1.0/24 |

### Firewall Policy

**Default Rule**: Deny all traffic

**Allow Rules Applied in Order:**
1. Anti-lockout rule (access to pfSense GUI)
2. VLAN-specific outbound rules
3. Inter-VLAN rules (explicit allow only)
4. VPN access rules

---

## Bandwidth Allocation

### QoS Strategy (Future Implementation)

| VLAN | Priority | Bandwidth Guarantee | Max Bandwidth |
|------|----------|---------------------|---------------|
| Management | High | 100 Mbps | Unlimited |
| Gaming | High | 500 Mbps | 1 Gbps |
| Services | Medium | 200 Mbps | 1 Gbps |
| Media | Medium | 200 Mbps | 1 Gbps |
| IoT | Low | 50 Mbps | 200 Mbps |
| Guest | Low | 50 Mbps | 200 Mbps |

---

## Failure Scenarios & Recovery

### Single Points of Failure

| Component | Impact | Mitigation | Recovery Time |
|-----------|--------|------------|---------------|
| pfSense | Total internet loss | Keep ISP router config, have spare hardware | <1 hour |
| Mokerlink Switch | Loss of VLAN routing | Use direct connections temporarily | <30 min |
| Proxmox Node 1 | Loss of primary VMs | Node 2 takes over via clustering | <5 min |
| ISP Router | Total internet loss | Contact ISP, use backup connection | 2-24 hours |

### Disaster Recovery Plan

1. **Configuration Backups**
   - pfSense: Weekly automated backups to cloud
   - Switch: Manual backup after changes
   - Proxmox: Daily VM backups to PBS

2. **Hardware Spares**
   - Keep old router as pfSense backup
   - Unmanaged switch can temporarily replace managed

3. **Documentation**
   - All configs in GitHub repository
   - Physical diagram posted in server room

---

## Design Decisions & Rationale

### Why DMZ Instead of Bridge Mode?

**Decision**: Use ISP router in DMZ mode, not bridge mode

**Rationale**:
- Household devices need to stay on ISP router network
- Bridge mode would disable ISP router's other ports
- DMZ provides isolation while maintaining household network
- Easier to troubleshoot (can bypass pfSense if needed)

**Trade-off**: Slight additional latency (~1ms) vs convenience

---

### Why VLAN Trunk Instead of Direct Port Assignment?

**Decision**: Use single trunk port from pfSense to managed switch

**Rationale**:
- Scalable - can add VLANs without rewiring
- Efficient use of pfSense ports (only 6 total)
- Industry standard approach
- Leaves pfSense ports available for future use (WiFi APs, additional switches)

**Trade-off**: Requires managed switch vs simplicity

---

### Why 10.0.x.x Instead of 192.168.x.x?

**Decision**: Use 10.0.0.0/8 address space

**Rationale**:
- More IP space (16 million vs 65k addresses)
- Clearer separation from household network (192.168.1.x)
- Professional/enterprise standard
- Room for growth (can segment 10.x.x.x by site/building later)

---

## Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| Internet throughput (no IDS) | 2.0 Gbps | ISP plan maximum |
| Internet throughput (with IDS/IPS) | 1.5 Gbps | 75% of line rate acceptable |
| Inter-VLAN latency | <1 ms | Hardware switching |
| WAN latency (to pfSense) | <2 ms | One additional hop from ISP router |
| VPN throughput | 1.0 Gbps | WireGuard on N355 |

---

## Future Enhancements

# VLAN Planning & Design

## VLAN Overview

This document details the Virtual LAN (VLAN) configuration for network segmentation and security isolation.

---

## VLAN Summary Table

| VLAN ID | Name | Subnet | Gateway | DHCP Range | Devices | Security Level |
|---------|------|--------|---------|------------|---------|----------------|
| **1** | Default/Native | N/A | N/A | N/A | Untagged management traffic | Infrastructure |
| **20** | Management | 10.0.20.0/24 | 10.0.20.1 | .100-.250 | Proxmox, switches, pfSense | **Trusted** |
| **30** | Services | 10.0.30.0/24 | 10.0.30.1 | .100-.250 | Docker, VMs, apps | Internal |
| **40** | Media | 10.0.40.0/24 | 10.0.40.1 | .100-.250 | Plex, Jellyfin, storage | Internal |
| **50** | IoT | 10.0.50.0/24 | 10.0.50.1 | .100-.250 | Smart devices, cameras | **Restricted** |
| **60** | Gaming/Clients | 10.0.60.0/24 | 10.0.60.1 | .100-.250 | Gaming PC, phones | Client |
| **61** | Red Team | 10.0.61.0/24 | 10.0.61.1 | .100-.250 | Security testing | **Isolated** |
| **62** | Purple Team | 10.0.62.0/24 | 10.0.62.1 | .100-.250 | Monitoring, IDS | Monitoring |
| **70** | Guest | 10.0.70.0/24 | 10.0.70.1 | .100-.250 | Visitor devices | **Restricted** |
| **99** | VPN | 10.0.99.0/24 | 10.0.99.1 | .10-.50 | Remote access | Internal |

---

## VLAN 20: Management

### Purpose
Critical infrastructure management and administration.

### Characteristics
- **Trust Level**: Highest (full network access)
- **Isolation**: Can access all other VLANs
- **Internet**: Full access
- **Performance**: Priority traffic

### Devices
- pfSense management interface
- Proxmox Node 1 (pve): 10.0.20.10
- Proxmox Node 2 (amdpve): 10.0.20.11
- Mokerlink managed switch: 10.0.20.2
- Zimaboard (future): 10.0.20.12
- Management laptop (when connected): 10.0.20.100+

### Access Control
**Allowed TO:**
- All VLANs (unrestricted)
- Internet (unrestricted)

**Allowed FROM:**
- VLAN 30 (Services) - SSH, Proxmox web (TCP 22, 8006)
- VLAN 62 (Purple Team) - Monitoring
- VPN VLAN 99 - Remote management

### Services Hosted
- Proxmox web interface (8006)
- SSH servers (22)
- Switch management web UI
- pfSense web GUI (443)

### Security Notes
- Physical access required for initial setup
- Two-factor authentication recommended
- Audit all connections
- Alert on unusual access patterns

---

## VLAN 30: Services

### Purpose
Self-hosted applications, containers, and general-purpose VMs.

### Characteristics
- **Trust Level**: Medium-High (internal services)
- **Isolation**: Controlled access to other VLANs
- **Internet**: Full access
- **Performance**: Standard

### Devices
- Docker host VMs
- Application VMs
- Development environments
- Web servers
- Database servers

### Planned Services
- Portainer (Docker management)
- Traefik (reverse proxy)
- Pi-hole / AdGuard Home (DNS filtering)
- Nextcloud (file sync)
- Vaultwarden (password manager)
- Home Assistant (if not on IoT)
- GitLab / Gitea (Git hosting)
- Uptime Kuma (monitoring)

### Access Control
**Allowed TO:**
- Internet (unrestricted)
- VLAN 20 (Management) - SSH, Proxmox API (specific ports)
- VLAN 40 (Media) - Full access (for media management)

**Allowed FROM:**
- VLAN 60 (Gaming/Clients) - HTTP/HTTPS only (TCP 80, 443)
- VLAN 99 (VPN) - Full access when authenticated

**Blocked TO/FROM:**
- VLAN 50 (IoT)
- VLAN 61 (Red Team)
- VLAN 70 (Guest)

### Firewall Rules
```
# Allow Services to Internet
Pass: SRVC net → any (! RFC1918)

# Allow Services to Media
Pass: SRVC net → MEDIA net

# Allow Services to Management (specific ports)
Pass: SRVC net → MGT net (TCP 22, 8006)

# Allow Clients to access Services web interfaces
Pass: GAMING net → SRVC net (TCP 80, 443)
```

---

## VLAN 40: Media

### Purpose
Media streaming, storage, and management.

### Characteristics
- **Trust Level**: Medium (media services)
- **Isolation**: Selective access
- **Internet**: Full access (metadata, updates)
- **Performance**: High throughput priority

### Devices
- Plex Media Server VM
- Jellyfin VM (alternative)
- Media storage VMs
- Download clients (Sonarr, Radarr, etc.)

### Services
- Plex (TCP 32400)
- Jellyfin (TCP 8096)
- Sonarr, Radarr, Lidarr
- Transmission / qBittorrent
- Media file storage (NFS/SMB shares)

### Access Control
**Allowed TO:**
- Internet (unrestricted)
- VLAN 30 (Services) - For management tools

**Allowed FROM:**
- VLAN 30 (Services) - Management/automation
- VLAN 60 (Gaming/Clients) - Streaming (TCP 32400, 8096)
- VLAN 99 (VPN) - Remote streaming

**Blocked TO/FROM:**
- VLAN 50 (IoT) - No smart TVs in IoT VLAN accessing media
- VLAN 61 (Red Team)
- VLAN 70 (Guest) - Guests cannot stream

### Storage Considerations
- Large file transfers (high bandwidth)
- Consider 10G networking in future
- NFS/SMB shares on VLAN 40

### Performance Optimization
- QoS priority for streaming traffic
- Direct connection to high-capacity storage
- Transcoding on GPU-enabled VMs

---

## VLAN 50: IoT (Internet of Things)

### Purpose
Smart home devices, cameras, and other IoT equipment.

### Characteristics
- **Trust Level**: LOW (untrusted devices)
- **Isolation**: **MAXIMUM** - Internet only, no LAN access
- **Internet**: Allowed (required for cloud services)
- **Performance**: Rate limited

### Devices
- Amazon Fire Stick
- Smart home hub
- Security cameras
- Smart lights, switches
- Smart thermostats
- Voice assistants
- Connected appliances

### Security Philosophy
**Zero Trust for IoT Devices**
- Assume all IoT devices are compromised
- Block all local network communication
- Allow only internet access
- Monitor for unusual traffic patterns

### Access Control
**Allowed TO:**
- **Internet ONLY** (! RFC1918 networks)

**Allowed FROM:**
- **NOTHING** (completely isolated)

**Explicitly Blocked:**
- All RFC1918 networks (10.x, 172.16.x, 192.168.x)
- Multicast / broadcast (prevents discovery)
- All other VLANs

### Firewall Rules
```
# Allow IoT to Internet ONLY
Pass: IOT net → any (! RFC1918, invert match)

# Block IoT to all local networks (explicit deny)
Block: IOT net → 10.0.0.0/8
Block: IOT net → 172.16.0.0/12
Block: IOT net → 192.168.0.0/16

# Block IoT from initiating any inbound
Block: any → IOT net
```

### Exceptions (If Needed)
If certain IoT devices MUST communicate with services (e.g., Home Assistant):

**Option 1**: Move Home Assistant to IoT VLAN
**Option 2**: Create specific firewall rule:
```
Pass: IOT net → SRVC net, destination 10.0.30.50 (Home Assistant), TCP 8123
```

### Monitoring
- Log all IoT traffic
- Alert on:
  - Attempts to access local networks
  - Unusual outbound destinations
  - High bandwidth usage
  - DNS queries for local hostnames

---

## VLAN 60: Gaming / Clients

### Purpose
Personal end-user devices (gaming PC, laptops, phones).

### Characteristics
- **Trust Level**: Medium (personal devices)
- **Isolation**: Selective access to services
- **Internet**: Full access
- **Performance**: Gaming traffic prioritized

### Devices
- Gaming PC: 10.0.60.50
- Laptops (when connected)
- Personal phones/tablets
- Workstations

### Access Control
**Allowed TO:**
- Internet (unrestricted)
- VLAN 30 (Services) - HTTP/HTTPS to web apps (TCP 80, 443)
- VLAN 40 (Media) - Streaming (TCP 32400, 8096)

**Allowed FROM:**
- VLAN 30 (Services) - Return traffic only

**Blocked TO:**
- VLAN 20 (Management) - Cannot manage infrastructure
- VLAN 50 (IoT) - Cannot access IoT devices
- VLAN 61 (Red Team)
- VLAN 62 (Purple Team)

### Services Accessed
- Self-hosted web apps (Nextcloud, etc.)
- Plex/Jellyfin streaming
- General internet browsing
- Gaming servers (external)

### QoS Configuration (Future)
- Prioritize gaming traffic (low latency)
- Bandwidth guarantee: 500 Mbps
- Max bandwidth: 1 Gbps

---

## VLAN 61: Red Team

### Purpose
Security testing, penetration testing, vulnerability assessment.

### Characteristics
- **Trust Level**: Isolated (assumed hostile)
- **Isolation**: Default deny, explicit allow only
- **Internet**: Allowed
- **Performance**: Standard

### Devices
- Raspberry Pi 4: 10.0.61.10
- Kali Linux VMs
- Security testing VMs
- Network scanning tools

### Use Cases
- Penetration testing practice
- Vulnerability scanning
- Security research
- CTF (Capture the Flag) training

### Access Control
**Allowed TO:**
- Internet (unrestricted)
- **Specific test targets only** (defined per-test)

**Allowed FROM:**
- VLAN 20 (Management) - SSH for control (TCP 22)

**Blocked BY DEFAULT:**
- All other VLANs (unless explicitly allowed for testing)

### Testing Methodology
**Before Testing:**
1. Define test scope (which VLAN/IP to test)
2. Add temporary firewall rule allowing Red Team → Target
3. Conduct testing
4. Remove firewall rule after testing
5. Document findings

**Example Test Rule** (temporary):
```
Pass: RED net → SRVC net, destination 10.0.30.100 (test VM)
Description: Security test - expires [date]
```

### Security Notes
- Never run production services on Red Team VLAN
- Treat as untrusted/compromised at all times
- Isolate from all other VLANs by default
- Log all activity

---

## VLAN 62: Purple Team (Monitoring)

### Purpose
Security monitoring, IDS analysis, log aggregation, metrics collection.

### Characteristics
- **Trust Level**: High (security monitoring)
- **Isolation**: Can monitor all VLANs (read-only)
- **Internet**: Allowed (updates, threat intelligence)
- **Performance**: Standard

### Devices
- Raspberry Pi 5: 10.0.62.10
- Monitoring VMs
- Log aggregation servers
- IDS analysis tools

### Services
- Grafana (visualization): TCP 3000
- Prometheus (metrics): TCP 9090
- Elasticsearch (logs)
- Suricata (IDS - on pfSense, data to Purple)
- Wazuh (SIEM)
- Uptime monitoring

### Access Control
**Allowed TO:**
- All VLANs (read-only monitoring)
- Internet (threat intelligence feeds)

**Allowed FROM:**
- VLAN 20 (Management) - Full access
- VLAN 30 (Services) - Push metrics/logs
- All VLANs - Push logs/metrics (UDP 514 syslog, etc.)

### Monitoring Capabilities
**Network Monitoring:**
- Suricata alerts from pfSense
- Flow data (NetFlow/IPFIX)
- DNS query logs
- Firewall logs

**System Monitoring:**
- Proxmox metrics
- VM/container performance
- Disk usage, CPU, RAM
- Service health checks

**Security Monitoring:**
- Failed login attempts
- Unusual traffic patterns
- Port scans from Red Team
- IoT unusual behavior

### Data Retention
- Real-time metrics: 7 days
- Aggregated metrics: 90 days
- Security logs: 1 year
- Critical alerts: Indefinite

---

## VLAN 70: Guest

### Purpose
Temporary access for visitors and untrusted devices.

### Characteristics
- **Trust Level**: Very Low (untrusted)
- **Isolation**: Internet only, no LAN access
- **Internet**: Allowed (rate limited)
- **Performance**: Low priority, bandwidth limited

### Devices
- Visitor laptops
- Guest phones
- Temporary contractors

### Access Control
**Allowed TO:**
- **Internet ONLY** (! RFC1918)

**Blocked TO:**
- All VLANs (complete isolation)

**Firewall Rules:**
```
# Allow Guest to Internet ONLY
Pass: GUEST net → any (! RFC1918)

# Block all local network access
Block: GUEST net → 10.0.0.0/8
```

### Bandwidth Limiting (Future QoS)
- Max bandwidth: 100 Mbps per device
- Total VLAN: 200 Mbps

### Captive Portal (Future)
- Login page with terms of service
- Time-limited access (4 hours, renewable)
- Device MAC tracking
- Voucher system for extended access

### Current Status
- **Not implemented yet**
- Low priority (no frequent guests)
- Will implement if needed

---

## VLAN 99: VPN

### Purpose
Remote access to homelab from external locations.

### Characteristics
- **Trust Level**: Medium-High (authenticated users)
- **Isolation**: Full access to internal VLANs
- **Internet**: Routed through homelab
- **Performance**: Limited by VPN throughput

### Devices
- WireGuard clients
- Phones (remote access)
- Laptops (while traveling)

### VPN Configuration
- **Protocol**: WireGuard
- **Server**: pfSense
- **Port**: UDP 51820
- **Encryption**: ChaCha20-Poly1305

### Access Control
**Allowed TO:**
- All internal VLANs (10.0.0.0/8)
- Internet (via homelab connection)

**Allowed FROM:**
- Internet (WireGuard handshake only)

### IP Assignments
- 10.0.99.1: VPN gateway
- 10.0.99.10-.50: DHCP pool for VPN clients

### Authentication
- Pre-shared keys (WireGuard)
- Per-device keys (rotate regularly)
- 2FA on critical services accessed via VPN

### Use Cases
- Remote Proxmox management
- Access self-hosted services while traveling
- Secure browsing on public WiFi
- Bypass geo-restrictions using home IP

---

## Switch Port VLAN Assignments

### Mokerlink Managed Switch

| Port | Mode | PVID | Tagged VLANs | Connected Device | Notes |
|------|------|------|--------------|------------------|-------|
| **1** | Trunk | 1 | 20,30,40,50,60,61,62,70 | pfSense Port 2 | All VLANs trunked |
| **2** | Access | 20 | - | Proxmox Node 1 (pve) | Management only |
| **3** | Access | 20 | - | Proxmox Node 2 (amdpve) | Management only |
| **4** | Access | 60 | - | Gaming PC | Client network |
| **5** | Access | 50 | - | NICGIGA Switch | IoT uplink |
| **6** | Access | 62 | - | Raspberry Pi 5 | Purple Team |
| **7** | Access | 61 | - | Raspberry Pi 4 | Red Team |
| **8** | Access | 20 | - | Available | Future expansion |

### NICGIGA Unmanaged Switch (All VLAN 50)

| Port | Connected Device | Notes |
|------|------------------|-------|
| **1** | Mokerlink Port 5 | Uplink (VLAN 50) |
| **2** | Amazon Fire Stick | IoT |
| **3** | Smart Home Hub | IoT |
| **4** | Available | Future IoT device |
| **5** | Available | Future IoT device |

---

## Proxmox VLAN Bridge Assignments

### Node 1 (pve) - 10.0.20.10
```bash
# /etc/network/interfaces

# Physical interface (no IP)
iface enp6s0 inet manual

# Management Bridge (VLAN 20) - untagged
auto vmbr0
iface vmbr0 inet static
    address 10.0.20.10/24
    gateway 10.0.20.1
    bridge-ports enp6s0
    bridge-stp off
    bridge-fd 0

# Services Bridge (VLAN 30)
auto vmbr30
iface vmbr30 inet manual
    bridge-ports enp6s0.30
    bridge-stp off
    bridge-fd 0

# Media Bridge (VLAN 40)
auto vmbr40
iface vmbr40 inet manual
    bridge-ports enp6s0.40
    bridge-stp off
    bridge-fd 0

# Red Team Bridge (VLAN 61)
auto vmbr61
iface vmbr61 inet manual
    bridge-ports enp6s0.61
    bridge-stp off
    bridge-fd 0
```

**When creating VMs:**
- Select `vmbr30` for Services VLAN
- Select `vmbr40` for Media VLAN
- Select `vmbr61` for Red Team testing
- Leave VLAN tag blank (already handled by bridge)

---

## VLAN Migration Plan

### Current State
- Flat network (192.168.x.x)
- No segmentation
- All devices on same broadcast domain

### Target State
- 8 isolated VLANs
- DMZ from household network
- Segmented by function and trust level

### Migration Steps

**Phase 1: Infrastructure (No Impact)**
1. Configure pfSense VLANs
2. Configure managed switch
3. Test connectivity between pfSense and switch

**Phase 2: Proxmox Migration (Minimal Impact)**
1. Migrate Proxmox Node 2 first (lower risk)
2. Verify connectivity
3. Migrate Proxmox Node 1
4. Update firewall rules

**Phase 3: Client Devices (User Impact)**
1. Move Gaming PC to VLAN 60
2. Test internet and service access
3. Move IoT devices to VLAN 50
4. Verify isolation

**Phase 4: Cleanup**
1. Remove old IP configurations
2. Disable old LAN interface on pfSense
3. Document final state

---

## Testing & Validation

### VLAN Connectivity Tests

**From each VLAN, test:**
```bash
# Test gateway
ping [VLAN gateway]

# Test internet
ping 8.8.8.8
ping google.com

# Test DNS
nslookup google.com

# Test allowed VLAN access
ping [allowed VLAN gateway]

# Test blocked VLAN access (should fail)
ping [blocked VLAN gateway]
```

### Security Validation

**IoT Isolation Test:**
```bash
# From IoT device (VLAN 50)
ping 10.0.20.1    # Should FAIL (Management)
ping 10.0.30.1    # Should FAIL (Services)
ping 10.0.60.100  # Should FAIL (Gaming)
ping 8.8.8.8      # Should SUCCEED (Internet)
```

**Red Team Isolation Test:**
```bash
# From Red Team (VLAN 61)
ping 10.0.20.1    # Should FAIL (no explicit rule)
ping 10.0.30.1    # Should FAIL (no explicit rule)
nmap 10.0.30.0/24 # Should be blocked (no explicit rule)
```

### Performance Tests

**Throughput:**
```bash
# iperf3 between VLANs (on allowed paths)
# Server on VLAN 30
iperf3 -s

# Client on VLAN 40 (allowed to access VLAN 30)
iperf3 -c 10.0.30.50

# Expected: ~2.3 Gbps (near line rate)
```

**Latency:**
```bash
# Ping between VLANs
ping 10.0.30.1 -c 100

# Expected: <1ms average
```

---

## Troubleshooting Guide

### Common Issues

**Issue**: Can't ping VLAN gateway
- Check physical cable
- Verify switch port PVID
- Check device IP configuration
- Verify VLAN exists on pfSense

**Issue**: Can access gateway but not internet
- Check firewall rules on pfSense
- Verify NAT configuration
- Check default gateway on device
- Test DNS resolution

**Issue**: Can't access other VLANs
- This is BY DESIGN for security
- Check if access should be allowed
- Add explicit firewall rule if needed
- Test rule application

**Issue**: IoT device can't reach cloud service
- Verify device has internet access
- Check if IoT firewall rule allows internet
- Ensure DNS is working
- Check for IP blocking (pfBlockerNG)

---

## Future VLAN Additions

### Potential VLANs

**VLAN 80: DMZ (Public Services)**
- Purpose: Services exposed to internet
- Subnet: 10.0.80.0/24
- Use case: Public web servers, game servers
- Security: Stricter rules, isolated from internal

**VLAN 90: Kubernetes**
- Purpose: Container orchestration network
- Subnet: 10.0.90.0/24
- Use case: K3s cluster networking
- Requirements: Separate from Docker VLAN

**VLAN 100-110: Lab/Testing**
- Purpose: Temporary testing environments
- Dynamic assignment
- Can be created/destroyed as needed

---

**Document Version**: 1.0  
**Last Updated**: 10/19/2025 
**Status**: Ready for Implementation  
**Next Review**: After migration completion
